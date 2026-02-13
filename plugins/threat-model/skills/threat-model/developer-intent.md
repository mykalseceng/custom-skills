# Phase 2: Developer Intent Analysis

Reconstruct what developers intended each function to do, map the assumptions holding the logic together, and find where code diverges from intent. This phase is the core innovation of the threat model — threats emerge from the gap between intent and reality.

**Prerequisite**: Phase 1 (Reconnaissance) must be complete. This phase uses workflows, state machines, and entry points from recon to decide which functions to analyze.

The output of this phase feeds Phase 3 (Logic Flaw Hunting) and Phase 4 (Threat Analysis). Cutting corners here guarantees hollow threats with no grounding in the actual code.

## Rationalizations to Reject

| Rationalization | Why It's Wrong | Required Action |
|----------------|---------------|-----------------|
| "The function name tells me the intent" | Names lie. `validateInput()` might only check length, not content. | Verify intent from 3 evidence sources |
| "Obviously the caller validates first" | "Obviously" = unvalidated assumption. Attackers love these. | Classify as UNVALIDATED until you verify the caller's code |
| "This assumption is safe because of the framework" | Frameworks have defaults, not guarantees. Configs override defaults. | Verify the framework assumption is enforced in THIS codebase |
| "Too many assumptions to list" | That's exactly when you need to list them. High assumption density = high threat density. | List them all; prioritize by trust boundary proximity |
| "This function is too simple for intent analysis" | Simple functions propagate assumptions silently. A 3-line helper that skips validation poisons every caller. | Analyze it, especially if it's called from multiple places |

---

## Developer Intent Reconstruction

For each function selected by the current depth level, reconstruct intent from three evidence sources. No single source is sufficient.

### Evidence Sources

| Source | What to Look For | Reliability |
|--------|-----------------|-------------|
| **Documented** | Docstrings, comments, API docs, README, commit messages, PR descriptions | Medium — docs can be stale or aspirational |
| **Structural** | Function name, parameter names/types, return types, module placement, error types defined | High — naming and structure reflect design-time intent |
| **Behavioral** | What the code actually does line-by-line, what it validates, what it rejects, what it returns on error | Highest — code is the ground truth |

### Intent Reconstruction Process

For each function:

1. **Read the documentation** (if any): docstring, inline comments, related API docs
2. **Read the structural signals**: function name, parameters, return type, where it lives in the module hierarchy, what calls it
3. **Read the code line-by-line**: what does it actually do? What does it validate? What does it skip?
4. **Synthesize intent**: write a 1-2 sentence intent statement
5. **Score alignment**: how well does the code match the intent?

### Per-Function Intent Template

```markdown
### `function_name` (file:line)

**Documented intent**: [from docstrings/comments, or "none"]
**Structural intent**: [from naming, types, module placement]
**Behavioral intent**: [from reading the actual code]

**Synthesized intent**: [1-2 sentences: what was this function meant to do?]

**Alignment**: [FULL / PARTIAL / DIVERGENT]
  - FULL: code achieves the intent completely
  - PARTIAL: code achieves most of the intent but misses edge cases or has gaps
  - DIVERGENT: code does something meaningfully different from the apparent intent

**Alignment gaps** (if PARTIAL or DIVERGENT):
  - [gap description] — [file:line evidence]
```

### Adaptive Depth — Which Functions to Analyze

| Depth | Functions to Analyze |
|-------|---------------------|
| **QUICK** | Entry point handlers + functions in critical workflows (from Phase 1 workflow inventory) |
| **STANDARD** | All public/exported functions + private functions on sensitive workflow paths |
| **DEEP** | Every non-trivial function including internal helpers and utilities |

**Promotion rule**: If a function initially classified as "not worth analyzing" is called from a sensitive path, promote it to full analysis. Never leave a gap in a critical workflow's function chain.

---

## Assumption Mapping

Every function rests on assumptions — about inputs, callers, environment, ordering, and trust. Most assumptions are implicit (not checked in code). The implicit ones are where threats live.

### 5 Assumption Types

#### 1. Input Assumptions

What the function assumes about its inputs.

```
Assumption: [description]
  Type: INPUT
  Parameter: [which parameter]
  Evidence: [file:line — validation code, type annotation, or lack thereof]
  Status: VALIDATED / UNVALIDATED / VIOLATED
  Validated by: [which code checks this — file:line, or "nothing"]
```

Examples:
- "user_id is a valid integer" — validated by type annotation, but not checked at runtime
- "email is well-formed" — validated by regex at `validators.py:23`
- "amount is positive" — UNVALIDATED: no check found anywhere in the call chain

#### 2. Caller Assumptions

What the function assumes about who calls it and what state they've established.

```
Assumption: [description]
  Type: CALLER
  Expected caller: [which function(s)]
  Evidence: [file:line]
  Status: VALIDATED / UNVALIDATED / VIOLATED
  Validated by: [access control, call chain analysis, or "nothing"]
```

Examples:
- "Only called after authentication middleware" — validated by middleware chain at `app.ts:15`
- "Caller has already checked permissions" — UNVALIDATED: no enforcement found
- "Request body has been parsed and validated by the time this runs" — check middleware ordering

#### 3. Environment Assumptions

What the function assumes about the runtime environment.

```
Assumption: [description]
  Type: ENVIRONMENT
  Dependency: [env var, config key, external service, file path]
  Evidence: [file:line]
  Status: VALIDATED / UNVALIDATED / VIOLATED
  Validated by: [startup check, config validation, or "nothing"]
```

Examples:
- "DATABASE_URL is set and points to a valid database" — validated by startup check at `config.py:8`
- "Redis is available and responding" — UNVALIDATED: no health check or fallback
- "Temp directory exists and is writable" — UNVALIDATED: assumed by `os.path.join(TEMP_DIR, ...)`

#### 4. Ordering Assumptions

What the function assumes about the order of operations.

```
Assumption: [description]
  Type: ORDERING
  Depends on: [which operation must happen first]
  Evidence: [file:line]
  Status: VALIDATED / UNVALIDATED / VIOLATED
  Validated by: [state checks, locks, or "nothing"]
```

Examples:
- "Payment is charged before order is created" — UNVALIDATED: no transactional guarantee
- "Token is validated before any database write" — validated by middleware ordering at `app.ts:12-15`
- "User profile exists before preferences are saved" — UNVALIDATED: race condition possible on new user flow

#### 5. Trust Assumptions

What the function assumes about the trustworthiness of data or callers.

```
Assumption: [description]
  Type: TRUST
  Trusted entity: [user input, other service, database query result, config]
  Evidence: [file:line]
  Status: VALIDATED / UNVALIDATED / VIOLATED
  Validated by: [validation code, or "nothing"]
```

Examples:
- "Data from the partner API is well-formed JSON" — UNVALIDATED: no schema validation on response
- "Admin users won't submit malicious input" — UNVALIDATED: no input sanitization on admin endpoints
- "Database query results are consistent" — assumption; concurrent writes could break this

---

## Intent-Reality Gap Analysis

After mapping intent and assumptions, identify where code diverges from intent. These gaps are the raw material for threat derivation in Phase 4.

### Common Gap Patterns

| Pattern | Description | Detection Signal |
|---------|-------------|-----------------|
| **Partial validation** | Function validates some inputs but not others | Some parameters have checks, others don't |
| **Happy-path-only error handling** | Function handles success but not failure modes | Missing catch blocks, unchecked error returns, no fallback |
| **Implicit coercion** | Function relies on language-level type coercion that changes semantics | String-to-number comparisons, truthiness checks on objects, `==` vs `===` |
| **Stale assumptions** | Code assumes a state that was true at design time but no longer holds | Comments referencing old behavior, dead code guards, version-specific workarounds |
| **Copy-paste drift** | Duplicated logic that's been updated in one place but not another | Similar functions with subtle differences in validation or error handling |
| **Over-trust of internal data** | Function treats database results or internal API responses as trusted without validation | No schema check on query results, no null check on joined data |
| **Missing terminal state** | State machine can reach a state from which no valid transition exists | State enums with values that no transition function produces or consumes |
| **Assumption chain break** | Function A assumes B validates; B assumes C validates; C assumes input is pre-validated | Trace the full chain — nobody actually validates |

### Gap Template

```markdown
### Gap: [short title]

**Function**: `function_name` (file:line)
**Intent**: [what was intended]
**Reality**: [what actually happens]
**Pattern**: [which common gap pattern, or "novel"]
**Broken assumption(s)**: [which assumption IDs from the mapping above]
**Potential impact**: [what could go wrong — to be refined in Phase 4]
```

---

## Cross-Function Assumption Chains

When Function A calls Function B, A's assumptions propagate to B. If B also makes assumptions that A doesn't know about, the chain can break silently.

### Chain Tracing

For each workflow identified in Phase 1:

1. List the functions in execution order
2. For each function boundary (A calls B):
   - What does A assume B will do?
   - What does B assume about its inputs (from A)?
   - Do A's guarantees match B's expectations?
3. Flag any break: where A's output doesn't satisfy B's input assumptions

### Broken Chain Template

```markdown
### Broken Chain: [workflow name]

**Chain**: A (file:line) → B (file:line) → C (file:line)

**Break point**: B → C
  - B outputs: [what B returns/sets]
  - C assumes: [what C expects]
  - Mismatch: [description of the gap]
  - Evidence: [file:line for both sides]
```

---

## Quality Thresholds

Minimums, not targets. Complex functions should exceed these.

| Metric | Minimum Per Function | Why |
|--------|---------------------|-----|
| Intent evidence sources used | 2 of 3 (documented, structural, behavioral) | Single source is unreliable |
| Assumptions documented | 5 | Fewer means you're taking things for granted |
| Assumptions classified (VALIDATED/UNVALIDATED/VIOLATED) | All documented assumptions | Unclassified assumptions are useless |
| Alignment scored | 1 per function | Forces explicit judgment |

### Global Minimums (across all analyzed functions)

| Metric | Minimum | Why |
|--------|---------|-----|
| UNVALIDATED assumptions identified | 10 | Every system has implicit assumptions; <10 means you weren't looking |
| Intent-reality gaps identified | 3 | If you found zero gaps, you didn't look hard enough |
| Cross-function assumption chains traced | 1 per workflow (from Phase 1) | Workflows are where chains break |
| Broken chains identified | 1 (at STANDARD+ depth) | At least one assumption chain is always broken |

---

## Completeness Gate

**Do NOT proceed to Phase 3 until every item below is satisfied.** If an item fails, go back and fix it.

### Structural Completeness

- [ ] All functions selected by the current depth level have been analyzed
- [ ] Every analyzed function has: intent statement, alignment score, and assumption list
- [ ] Every assumption is classified as VALIDATED / UNVALIDATED / VIOLATED

### Content Depth

- [ ] Per-function quality thresholds met (evidence sources, assumptions, classification)
- [ ] Global quality thresholds met (UNVALIDATED count, gaps, chains, broken chains)
- [ ] No function analysis contains vague language ("probably", "likely", "should") without an explicit "Unclear; need to inspect X" marker

### Integration

- [ ] Cross-function assumption chains traced for all Phase 1 workflows
- [ ] Broken chains documented with evidence from both sides
- [ ] Phase 1 recon output (workflows, state machines, entry points) was used to guide function selection

### Anti-Hallucination

- [ ] Every assumption cites a file:line reference or is marked as inferred
- [ ] Every VIOLATED classification has file:line evidence of the violation
- [ ] No intent statement contradicts the behavioral evidence (code)

**If any checkbox fails**: go back and fix it before proceeding to Phase 3.

---

## Phase 2 Output Template

```markdown
## Developer Intent Summary

### Functions Analyzed: [count]
### Depth: [QUICK / STANDARD / DEEP]
### Completeness Gate: [PASSED / FAILED — list failures]

### Alignment Summary
| Alignment | Count | Percentage |
|-----------|-------|-----------|
| FULL | [n] | [%] |
| PARTIAL | [n] | [%] |
| DIVERGENT | [n] | [%] |

### Assumption Inventory
| # | Assumption | Type | Function | Status | Evidence |
|---|-----------|------|----------|--------|----------|
| A-001 | user_id is valid integer | INPUT | get_user() | UNVALIDATED | user_service.py:42 |
| A-002 | caller authenticated | CALLER | update_profile() | VALIDATED | middleware.ts:15 |

### Key Assumption Stats
- Total assumptions mapped: [n]
- VALIDATED: [n] ([%])
- UNVALIDATED: [n] ([%])
- VIOLATED: [n] ([%])

### Intent-Reality Gaps
| # | Gap | Function | Pattern | Broken Assumptions |
|---|-----|----------|---------|-------------------|
| G-001 | [title] | func() at file:line | Partial validation | A-001, A-003 |

### Broken Assumption Chains
| # | Workflow | Chain | Break Point | Mismatch |
|---|---------|-------|-------------|----------|
| C-001 | Payment | charge() → create_order() → notify() | charge → create_order | No transactional guarantee |

### Signals for Phase 3 (Logic Flaw Hunting)
1. [signal] — [which flaw categories to prioritize]
2. ...

### Unresolved Questions
1. [question] — blocked by [what would answer it]
```
