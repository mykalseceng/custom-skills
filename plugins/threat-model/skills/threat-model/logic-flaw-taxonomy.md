# Phase 3: Logic Flaw Hunting

Systematically hunt for logic flaws using 12 categories derived from the assumption inventory and intent-reality gaps from Phase 2. This is reasoning-based analysis, not grep-based pattern matching.

**Prerequisite**: Phase 2 (Developer Intent Analysis) must be complete with the completeness gate passed. This phase uses the assumption inventory, intent-reality gaps, and broken assumption chains to guide where to look.

---

## 12 Logic Flaw Categories

For each category: understand the definition, apply the detection approach against the codebase (guided by Phase 2 outputs), and document findings.

### 1. Authorization Logic Gaps

**Definition**: The system checks *whether* a user is authenticated but fails to check *what* they're authorized to do, or checks authorization inconsistently across code paths.

**Detection approach**:
- Review every CALLER assumption of type "caller has permission X" from Phase 2
- For each protected endpoint: is authorization checked, or only authentication?
- Compare parallel code paths (e.g., GET vs PUT on same resource) — are authz checks consistent?
- Look for direct object references: does the code verify the requesting user owns the resource?

**Common manifestations**:
- IDOR: `get_order(order_id)` without checking `order.user_id == current_user.id`
- Role check at route level but not at service level (bypass via internal API)
- Admin-only function callable by anyone who knows the endpoint

**Exploitability signal**: HIGH if any UNVALIDATED CALLER assumption involves authorization

### 2. State Machine Violations

**Definition**: State transitions occur that the state machine design didn't intend — skipping states, reaching invalid states, or transitions without proper guards.

**Detection approach**:
- Use Phase 1 state machine inventory
- For each transition: is the guard condition checked? Can the guard be bypassed?
- Look for direct status field writes (`order.status = 'completed'`) that bypass transition functions
- Check: can any state be reached from an unexpected predecessor?

**Common manifestations**:
- Order goes from `pending` to `shipped` without passing through `paid`
- Account `suspended` but user can still perform actions (status checked in some paths but not others)
- No terminal state enforcement — completed orders can be re-opened

**Exploitability signal**: HIGH if state machines lack centralized transition logic

### 3. Race Conditions / TOCTOU

**Definition**: A time-of-check-to-time-of-use gap where the state can change between when it's checked and when it's acted upon.

**Detection approach**:
- Review ORDERING assumptions from Phase 2 — any "check then act" patterns without atomicity?
- Look for: read-modify-write without locks, check-then-update without transactions, file existence checks followed by file operations
- Identify shared mutable state accessed by concurrent code paths

**Common manifestations**:
- Balance check → deduct pattern without database transaction (double-spend)
- Permission check → action without holding lock (escalation between check and action)
- File exists check → read file (symlink race)

**Exploitability signal**: HIGH if the system handles concurrent requests to the same resource

### 4. Missing Edge Cases

**Definition**: Code handles the common case correctly but fails on boundary values, empty inputs, overflow, underflow, or unusual-but-valid inputs.

**Detection approach**:
- For each INPUT assumption from Phase 2 with status UNVALIDATED: what happens at the boundary?
- Check: zero, negative, empty string, empty array, null, MAX_INT, unicode, very long strings
- Look for off-by-one errors in loops, ranges, and array indexing
- Check division operations for zero divisors

**Common manifestations**:
- Pagination with `page=0` or `page=-1` returns unexpected data
- Username validation allows empty string
- Quantity field allows negative values (credit instead of charge)

**Exploitability signal**: MEDIUM-HIGH depending on what the edge case unlocks

### 5. Order-of-Operations Errors

**Definition**: Operations execute in the wrong order, or the code assumes an order that isn't guaranteed.

**Detection approach**:
- Review ORDERING assumptions from Phase 2
- Check middleware/interceptor ordering — is it explicitly defined or implicit?
- Look for "setup before use" patterns where setup might not have completed
- Check event handler registration order

**Common manifestations**:
- Auth middleware registered after route handler (auth bypassed on early routes)
- Validation runs after database write (invalid data persisted)
- Cleanup runs before the operation it's supposed to clean up after

**Exploitability signal**: HIGH if ordering affects auth/authz/validation

### 6. Implicit Type Coercion / Conversion

**Definition**: Language-level implicit type conversion changes the semantics of a comparison or operation in ways the developer didn't intend.

**Detection approach**:
- Language-specific: `==` vs `===` in JavaScript, truthy/falsy confusion, string-to-number coercion
- Look for comparisons between different types
- Check JSON parsing: does the parser enforce expected types?
- Look for integer overflow in languages without automatic big-integer promotion

**Common manifestations**:
- `if (user.role == 0)` in JS treats `null`, `undefined`, `""`, `false` as zero
- String comparison of version numbers: `"9" > "10"` is true lexicographically
- Integer truncation when assigning to a smaller type

**Exploitability signal**: MEDIUM — requires understanding of language-specific type system

### 7. Error Handling Logic Flaws

**Definition**: Error handling code itself contains logic errors, or error paths lead to insecure states.

**Detection approach**:
- Review all intent-reality gaps with pattern "Happy-path-only error handling" from Phase 2
- For each error path: does the system end up in a consistent state?
- Check: are partial operations rolled back on failure?
- Look for catch-all exception handlers that swallow errors silently

**Common manifestations**:
- Payment deducted but order creation fails — no refund logic
- Error handler returns a default value that bypasses security checks
- Exception caught and logged but execution continues with stale/invalid data

**Exploitability signal**: HIGH if error paths handle sensitive operations

### 8. Business Logic Bypass

**Definition**: The business rules intended by stakeholders are not fully encoded in the code, allowing users to circumvent intended constraints.

**Detection approach**:
- For each workflow from Phase 1: are all business rules enforced at the code level?
- Look for constraints expressed only in UI/frontend (not enforced server-side)
- Check discount/coupon/pricing logic for edge cases (negative prices, stacked discounts, currency rounding)
- Check rate limits, usage quotas, trial period enforcement

**Common manifestations**:
- Coupon applied multiple times (no server-side single-use enforcement)
- Free trial restartable by re-registering with same email
- Price calculated client-side and trusted by server

**Exploitability signal**: HIGH — business logic bypasses are often trivially exploitable

### 9. Trust Boundary Confusion

**Definition**: The code treats data from an untrusted source as if it came from a trusted source, or misidentifies which side of a trust boundary something is on.

**Detection approach**:
- Review all TRUST assumptions from Phase 2 with status UNVALIDATED
- For each external data source (API responses, database reads after user writes, file reads): is the data validated after crossing the boundary?
- Check: does the code re-validate data that was stored earlier by untrusted input?

**Common manifestations**:
- Admin dashboard reads user-submitted data and renders it without sanitization (stored XSS via trust confusion)
- Internal service trusts data from another service that accepts user input
- Cache returns stale data that was valid when cached but is now dangerous

**Exploitability signal**: HIGH if trust boundaries involve user-controlled data

### 10. Assumption Propagation Failures

**Definition**: Function A makes an assumption, passes data to B assuming B shares the same assumption, but B has different (or no) assumptions about that data.

**Detection approach**:
- Use the broken assumption chains from Phase 2 directly
- For each chain break: what's the worst-case scenario if the assumption fails?
- Look for wrapper functions that strip context (e.g., a wrapper catches all errors, hiding which specific error occurred from the caller)

**Common manifestations**:
- Validation function returns boolean but caller doesn't check the return value
- Sanitization function strips some dangerous chars but not all; downstream function assumes full sanitization
- Permission check in controller but service layer assumes caller is authorized

**Exploitability signal**: HIGH — this is where multi-step attacks are built

### 11. Fail-Open vs Fail-Secure

**Definition**: When a security check encounters an error or unexpected condition, it defaults to allowing access (fail-open) instead of denying it (fail-secure).

**Detection approach**:
- For each auth/authz function: what happens when the check throws an exception?
- For each validation function: what's the default return on error?
- Check middleware: if middleware fails, does the request proceed or get rejected?
- Look for `catch` blocks that return `true` or allow execution to continue

**Common manifestations**:
- JWT verification catches exception → returns `{valid: true}` as default
- Rate limiter fails to connect to Redis → allows all requests through
- Feature flag service unavailable → all features enabled (including admin features)

**Exploitability signal**: CRITICAL — attackers can trigger the failure condition intentionally

### 12. Temporal Logic Flaws

**Definition**: Logic that depends on time (expiration, cooldowns, rate windows, scheduling) has implementation errors.

**Detection approach**:
- Review all assumptions involving time, timestamps, expiration, TTL
- Check: are time comparisons using consistent units (seconds vs milliseconds)?
- Look for: timezone confusion, clock skew assumptions, missing expiration checks
- Check: can tokens/sessions be used after expiration due to caching?

**Common manifestations**:
- Token expiration checked in seconds but stored in milliseconds (effectively never expires)
- Rate limiter uses wall clock — clock adjustment bypasses the limit
- Password reset token has no expiration, or expiration isn't enforced

**Exploitability signal**: MEDIUM-HIGH depending on what the temporal logic protects

---

## Adversarial Workflow Reconstruction

For each workflow from Phase 1, walk through it as three adversarial personas. Each persona targets different flaw categories.

### Personas

| Persona | Mindset | Targets |
|---------|---------|---------|
| **The Scoundrel** | "How do I get something I shouldn't have?" | Authorization gaps (#1), business logic bypass (#8), trust boundary confusion (#9) |
| **The Confused Developer** | "What if I use this in a way that seems valid but isn't intended?" | State machine violations (#2), missing edge cases (#4), implicit coercion (#6) |
| **The Opportunist** | "What if something goes wrong at just the right time?" | Race conditions (#3), error handling flaws (#7), fail-open (#11), temporal flaws (#12) |

### Per-Workflow Walkthrough

For each workflow and each persona:

```markdown
### Workflow: [name] as [Persona]

**Goal**: [what this persona is trying to achieve]
**Entry point**: [where they start]

**Steps**:
1. [action] → [expected system response] → [persona's exploitation attempt]
2. [action] → [expected response] → [exploitation attempt]
3. ...

**Flaws found**:
- [Flaw category #N]: [description] — [file:line]

**Flaws NOT found** (checked but clear):
- [Flaw category #N]: [why it's not exploitable here]
```

**Important**: Document what you checked and found clear, not just what you found broken. This proves coverage and prevents revisiting.

---

## Compound Flaws

After individual flaw hunting, look for combinations where two or more flaw categories interact to create a more severe issue.

### Common Compound Patterns

| Flaw A | + Flaw B | = Compound Threat |
|--------|---------|------------------|
| Authorization gap (#1) | + Race condition (#3) | Escalate privileges by racing a role change |
| State machine violation (#2) | + Missing edge case (#4) | Reach an invalid state that unlocks hidden functionality |
| Error handling flaw (#7) | + Fail-open (#11) | Trigger an error to bypass a security check |
| Trust boundary confusion (#9) | + Assumption propagation (#10) | Inject trusted-looking data that propagates unchecked |
| Business logic bypass (#8) | + Temporal flaw (#12) | Exploit expired but cached authorization to bypass business rules |

### Compound Flaw Template

```markdown
### Compound: [title]

**Component flaws**:
- [Category #N]: [description] at [file:line]
- [Category #M]: [description] at [file:line]

**Combined effect**: [what the combination enables that neither flaw enables alone]
**Attack scenario**: [concrete multi-step exploit]
```

---

## Phase 3 Output Template

```markdown
## Logic Flaw Summary

### Flaw Inventory
LF-NNN IDs are working notation for this phase. Each flaw maps to an F-NNN finding in the report.

| # | Category | Flaw | Location | Exploitability | Related Assumptions |
|---|---------|------|----------|---------------|-------------------|
| LF-001 | #1 Authorization Gap | [description] | file:line | HIGH | A-001, A-005 |
| LF-002 | #2 State Machine | [description] | file:line | MEDIUM | A-012 |

### Flaw Stats by Category
| Category | Flaws Found | Highest Exploitability |
|----------|------------|----------------------|
| #1 Authorization Logic Gaps | [n] | [level] |
| #2 State Machine Violations | [n] | [level] |
| ... | | |

### Adversarial Walkthrough Coverage
| Workflow | Scoundrel | Confused Dev | Opportunist | Flaws Found |
|---------|-----------|-------------|-------------|-------------|
| [name] | DONE | DONE | DONE | [count] |

### Compound Flaws
| # | Component Flaws | Combined Effect | Severity Uplift |
|---|----------------|----------------|----------------|
| CF-001 | LF-001 + LF-003 | [effect] | MEDIUM → HIGH |

### Signals for Phase 4 (Threat Analysis)
1. [signal] — [which threats to derive from which flaws/assumptions]
2. ...
```
