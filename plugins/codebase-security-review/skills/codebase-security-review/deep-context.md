# Phase 2: Deep Context Building

Build line-by-line understanding of the code before making any security judgments. This phase is pure comprehension — no findings, no verdicts, no severity ratings yet.

**Prerequisite**: Phase 1 (Reconnaissance) must be complete. This phase uses the recon output — entry points, auth map, dependency inventory, framework detection — to decide which functions to analyze and at what depth.

The output of this phase feeds every subsequent phase. Cutting corners here guarantees hallucinated findings later.

Three sub-phases, always in order:

```
Sub-Phase 2A: Orientation    → Import recon output, supplement anchors
Sub-Phase 2B: Deep Analysis  → Line-by-line micro-analysis of selected functions
Sub-Phase 2C: Synthesis      → Extract invariants, map state, identify risk clusters
```

---

## Rationalizations (Do Not Skip)

| Rationalization | Why It's Wrong | Required Action |
|-----------------|----------------|-----------------|
| "I get the gist" | Gist-level understanding misses edge cases and off-by-ones | Line-by-line analysis required |
| "This function is simple" | Simple functions compose into complex bugs | Apply 5 Whys anyway |
| "I'll remember this invariant" | You won't. Context degrades across phases. | Write it down explicitly |
| "External call is probably fine" | External = adversarial until proven otherwise | Jump into code or model as hostile |
| "I can skip this helper" | Helpers contain assumptions that propagate silently | Trace the full call chain |
| "This is taking too long" | Rushed context = hallucinated vulnerabilities later | Slow is fast |
| "I already know this pattern" | Familiarity breeds blind spots; you skip what feels obvious | Analyze it anyway |

---

## Sub-Phase 2A: Orientation

Import the Phase 1 (Reconnaissance) output and supplement with anything recon missed. Do NOT re-discover what recon already found — build on it.

### Steps

1. **Import recon output.** Copy the entry points, auth map, dependency inventory, language/framework detection, and secret findings from Phase 1. These are your starting anchors.

2. **Supplement modules.** If recon didn't map all major modules/packages, fill gaps now. Add one-line descriptions of apparent purpose.

3. **Identify actors.** Who interacts with this system? Users, admins, other services, cron jobs, external APIs. List with their apparent privilege level. (Recon identified entry points; now identify who uses them.)

4. **Identify important state.** Global variables, singletons, caches, database tables/collections, session stores, config objects. List with their location. (Not covered by recon.)

5. **Note anything surprising.** Unusual patterns, unexpected binaries, vendored code, generated code, discrepancies between recon findings and actual code. Flag but don't investigate yet.

### Orientation Output

```
## 2A: Orientation Anchors

[Imported from Phase 1 Recon]
  Entry Points: [count] (from recon)
  Auth Map: [summary] (from recon)
  Dependencies: [count] (from recon)
  Languages/Frameworks: [list] (from recon)

[Supplemented in this phase]
  Modules: [count]
    - [module] → [apparent purpose]
  Actors: [count]
    - [actor] → [privilege level]
  State: [count]
    - [variable/table] → [location]
  Surprises: [list anything unexpected]
```

This output establishes anchors for Sub-Phase 2B. Refer back to it constantly during deep analysis.

---

## Sub-Phase 2B: Deep Analysis

Line-by-line micro-analysis of functions selected by the current depth level. This is the core of context building. Use Phase 1 recon output to prioritize: start with entry points that handle auth, user input, or sensitive data.

### Adaptive Depth — Which Functions to Analyze

| Depth | Functions to Analyze |
|-------|---------------------|
| **QUICK** | Entry points (route handlers, CLI commands, message handlers) + functions they directly call that handle auth, crypto, or user input |
| **STANDARD** | All public/exported functions + private functions called from public APIs that handle sensitive operations |
| **DEEP** | Every non-trivial function including internal helpers, utility functions, and test setup code |

**Promotion rule**: If a function initially classified as "not worth analyzing" turns out to be called from a security-critical path, promote it and analyze it fully. Never leave a gap in a critical call chain.

### Per-Function Micro-Analysis

For each function selected by the current depth level:

#### 1. Purpose

State in one sentence WHY this function exists. Not what it does — why it needs to exist.

#### 2. Inputs & Assumptions

- **Explicit inputs**: Parameters, their types, expected ranges
- **Implicit inputs**: Global state, environment variables, config, time, randomness
- **Preconditions**: What must be true before this function runs? Who enforces it — caller or callee?
- **Trust level**: Are inputs from trusted internal code or untrusted external sources?

#### 3. Outputs & Effects

- **Return values**: What does it return? What do different return values mean?
- **State mutations**: What global/shared state does it write?
- **External calls**: What external systems does it touch (DB, network, filesystem)?
- **Events/signals**: Does it emit events, publish messages, trigger callbacks?
- **Error behavior**: What happens on failure? Does it throw, return null, return partial data, panic?

#### 4. Block-by-Block Analysis

For each logical block within the function:

```
Block: [line range]
  What:       [What this block does]
  Why here:   [Why it's at this position — what depends on it running before/after]
  Assumes:    [What conditions this block relies on]
  Maintains:  [What invariants this block preserves or establishes]
  Depends on: [What later logic relies on this block's output]
```

Apply per-block:

- **First Principles**: What is the most fundamental thing this block does? Could it be wrong even if it looks correct? Strip away assumptions — what does the code *actually* do vs what it *appears* to do?
- **5 Whys** (on anything surprising): Why is this check here? → Because input could be null → Why could input be null? → Because the caller doesn't validate → Why doesn't the caller validate? → ...
- **5 Hows** (on state transitions): How does this value get set? → Via config load → How is config loaded? → From env vars → How are env vars set? → ...

#### 5. Function-Level Summary

```markdown
### `function_name` (file:line)

**Purpose**: [One sentence]
**Trust boundary**: [Internal / External-facing / Mixed]
**Risk signal**: [LOW / MEDIUM / HIGH — based on input trust + effects]

**Inputs**: [param types + trust levels]
**Outputs**: [returns + side effects]
**Invariants maintained**: [What must remain true — minimum 3]
**Invariants assumed**: [What it relies on from callers]
**Key assumptions**: [Anything non-obvious — minimum 5 per function]
**External interactions**: [Risk considerations — minimum 3 if function has external calls]
**First Principles applied**: [At least 1 per function]
**5 Whys / 5 Hows applied**: [At least 3 combined per function]
```

### Cross-Function Flow Tracing

When a function calls another function, trace the data and assumptions across the boundary. **Continuity rule**: Treat the entire call chain as one continuous execution flow. Never reset your mental model when crossing a call boundary. All invariants, assumptions, and data dependencies must propagate across calls.

#### Internal Calls (code available)

1. Jump into the callee — apply the same micro-analysis
2. Track what happens to the caller's data inside the callee
3. Track what assumptions the callee makes about its inputs
4. Verify: does the caller satisfy those assumptions?
5. Track what the callee returns and how the caller uses it

#### External Calls (code available — library/dependency in codebase)

Treat as internal calls. Jump into the library source. Continue block-by-block micro-analysis. Propagate invariants and assumptions seamlessly. Consider edge cases based on the *actual* code, not a black-box guess.

#### External Calls (black box — no source available)

**External = adversarial until proven otherwise.** Model the callee as hostile:

- It may return unexpected types, null, empty collections, negative numbers
- It may throw exceptions the caller doesn't catch
- It may take arbitrarily long (timeout?)
- It may call back into the caller's code (reentrancy)
- It may mutate shared state the caller doesn't expect
- It may succeed but produce semantically wrong results
- It may return valid-looking data that is semantically incorrect

Document:
- What the caller **assumes** about the external call
- What the external call **guarantees** (from docs/contracts)
- The **gap** between assumptions and guarantees

### Subagent Usage

Spawn subagents for parallel analysis when encountering:

- **Dense or complex functions** (>50 lines with multiple branches, state mutations, and external calls)
- **Long call chains** (>3 hops deep where carrying context manually risks loss)
- **Cryptographic or mathematical logic** (requires focused attention without surrounding noise)
- **Complex state machines** (multiple states, transitions, and edge cases)
- **Multi-module workflow reconstruction** (tracing a flow across 4+ files)

Subagents **must** follow the same micro-analysis rules. Return structured summaries that integrate into the global model. Never accept a subagent's summary without verifying it connects to your existing invariants and assumptions.

---

## Sub-Phase 2C: Synthesis

After sufficient micro-analysis, step back and build the global picture. This is where individual function understanding becomes system understanding.

### 1. Invariant Extraction

#### Must-Always-Be-True Invariants

Things that must hold at all times for the system to be correct:
- "User ID in session must correspond to an active user in the database"
- "Balance can never be negative"
- "Auth middleware runs before any route handler on protected paths"

#### Must-Never-Happen Invariants

Things that must never occur:
- "Unauthenticated request must never reach admin endpoints"
- "User input must never be interpolated into SQL without parameterization"
- "Secret key must never appear in logs or error messages"

#### Cross-Module Invariants

Invariants that span multiple files or modules:
- "Every write to the orders table must be preceded by a balance check in the payments module"
- "Rate limiter in middleware must be consistent with the rate limit config in settings"

### 2. State & Trust Mapping

#### State Variable Map

For each stateful variable (globals, singletons, caches, sessions, DB tables):

```
Variable: [name]
  Defined: [file:line]
  Writers: [list of functions that modify it]
  Readers: [list of functions that read it]
  Trust:   [Who can influence its value? Internal only / External-influenced]
  Race:    [Can concurrent access cause inconsistency?]
```

#### Trust Boundary Map

```
Boundary: [name, e.g., "HTTP request → route handler"]
  Untrusted side: [What's on the untrusted side]
  Trusted side:   [What's on the trusted side]
  Validation:     [What validation happens at this boundary]
  Gaps:           [What's NOT validated]
```

### 3. Workflow Reconstruction

Identify end-to-end flows and trace how state transforms across them:

- **User-facing workflows**: login → session creation → authenticated request → logout
- **Data lifecycle**: creation → validation → storage → retrieval → transformation → output
- **Admin/privileged workflows**: config change → propagation → effect
- **Error/recovery flows**: failure → retry → fallback → logging

For each workflow, record:
- The sequence of functions involved
- State transformations at each step
- Assumptions that persist across steps
- Where the workflow can be interrupted or diverted

### 4. Risk Clustering

Identify areas where multiple risk signals converge. These clusters are where vulnerabilities are most likely to hide:

- Untrusted input + missing validation + state mutation
- External call + no error handling + state change on success assumed
- Auth check in caller + no re-check in callee + privilege-sensitive operation
- Concurrent access + shared mutable state + no locking
- User-controlled path + file operation + no canonicalization

---

## Quality Thresholds

These are minimums, not targets. Exceeding them is expected for complex functions.

| Metric | Minimum Per Function | Why |
|--------|---------------------|-----|
| Invariants documented | 3 | Fewer means you haven't understood the function's contract |
| Assumptions documented | 5 | Fewer means you're taking things for granted |
| Risk considerations for external interactions | 3 (if applicable) | External calls are the #1 source of missed bugs |
| First Principles applications | 1 | Forces you to question the obvious |
| 5 Whys / 5 Hows applications (combined) | 3 | Forces depth on surprising or stateful logic |
| Block-by-block entries | 1 per logical block | Skipping blocks = skipping understanding |

### Global Minimums (across all analyzed functions)

| Metric | Minimum | Why |
|--------|---------|-----|
| Must-always-be-true invariants | 5 | Every system has core correctness properties |
| Must-never-happen invariants | 5 | Every system has safety boundaries |
| Cross-module invariants | 3 (if multi-module) | Module boundaries are where assumptions break |
| Trust boundaries identified | 2 | Every system has at least internal/external |
| State variables of concern | 3 | Stateful bugs are the hardest to find later |
| Risk clusters identified | 2 | If you found zero clusters, you weren't looking |

---

## Completeness Gate

**Do NOT proceed to Phase 3 until every item below is satisfied.** If an item fails, go back and fix it. This gate exists because incomplete context guarantees hallucinated findings.

### Structural Completeness

- [ ] All required sub-phases completed (2A Orientation, 2B Deep Analysis, 2C Synthesis)
- [ ] All functions selected by the current depth level have been analyzed
- [ ] Every analyzed function has all 5 sections (Purpose, Inputs, Outputs, Block-by-Block, Summary)
- [ ] Cross-function flow tracing done for every call from an analyzed function to another

### Content Depth

- [ ] Per-function quality thresholds met (invariants, assumptions, risk considerations, First Principles, 5 Whys/Hows)
- [ ] Global quality thresholds met (system invariants, trust boundaries, state variables, risk clusters)
- [ ] No function summary contains vague language ("probably", "likely", "should", "seems to") without an explicit "Unclear; need to inspect X" marker
- [ ] Phase 1 recon output was imported and used to guide function selection

### Continuity & Integration

- [ ] Cross-references exist between functions that share state or assumptions
- [ ] Invariants from Sub-Phase 2B are reflected in Sub-Phase 2C extraction
- [ ] Trust boundary map in 2C is consistent with per-function trust level assessments in 2B
- [ ] No "island" functions — every analyzed function connects to at least one other through calls, shared state, or shared invariants

### Anti-Hallucination

- [ ] Every claim cites a file:line reference or is explicitly marked as inferred
- [ ] No invariant is stated without identifying which functions maintain it and which assume it
- [ ] All "unclear" items from 2B are either resolved in 2C or listed in Unresolved Questions
- [ ] Mental model was updated at least once during analysis (if it wasn't, you weren't paying attention)

**If any checkbox fails**: go back to the relevant sub-phase and fix it before proceeding.

---

## Anti-Hallucination Rules

These rules are mandatory throughout all sub-phases. Violating them invalidates the review.

1. **Never reshape evidence to fit earlier assumptions.** If code contradicts your mental model, update the model, don't explain away the code.
2. **Update explicitly when contradicted.** Say: "Earlier I assumed X. Line Y:Z shows X is false. Updated model: ..."
3. **"Unclear; need to inspect X" is always valid.** Never say "it probably does Y" without reading the code.
4. **Cross-reference constantly.** If function A assumes B validates input, verify B actually validates input. Don't assume.
5. **Track uncertainty.** Mark assumptions that haven't been verified yet. Come back to them.
6. **Distinguish code-verified from inferred.** "Code at file:line confirms X" vs "I infer X based on naming convention."
7. **Anchor periodically.** After every 3-5 functions, re-summarize your current understanding of core invariants, state relationships, and actor roles. This prevents drift.

---

## Phase 2 Output Template

After completing all three sub-phases, produce:

```markdown
## Deep Context Summary

### Functions Analyzed: [count]
### Depth: [QUICK / STANDARD / DEEP]
### Completeness Gate: [PASSED / FAILED — if failed, list which items]

### Orientation Anchors (imported from Phase 1 + supplemented)
- Entry points: [count] (from recon)
- Modules: [count]
- Actors: [list]
- Key state: [list]

### Key Invariants
#### Must-Always-Be-True
1. [invariant] — maintained by [functions], assumed by [functions]
2. ...

#### Must-Never-Happen
1. [invariant] — enforced by [functions], violated if [condition]
2. ...

#### Cross-Module
1. [invariant] — spans [modules], coupling point at [file:line]
2. ...

### Trust Boundaries Identified
1. [boundary] — validation: [present/partial/missing] — gaps: [list]
2. ...

### State Variables of Concern
1. [variable] — [why: race condition / external influence / missing validation / multiple writers]
2. ...

### Risk Clusters
Areas where multiple risk signals converge:
1. [area] — [signals: untrusted input + missing validation + state mutation + ...]
2. ...

### Key Workflows Traced
1. [workflow] — [sequence of functions] — [key risk points]
2. ...

### Unresolved Questions
1. [question] — blocked by [what would answer it]
2. ...

### Model Updates During Analysis
1. "Earlier assumed [X]. File:line showed [Y]. Updated to [Z]."
2. ...
```

This output feeds directly into Phase 3 (Threat Model).
