# Phase 5: Report

Template and guidelines for producing the final threat model document. The report synthesizes all phases into a single, ranked document that communicates threats clearly and actionably.

**This phase MUST produce a written markdown file.** The report is the deliverable — analysis without a written report is incomplete.

---

## Report File

File name: `<PROJECT>_THREAT_MODEL_<YYYY-MM-DD>.md`

Write the report to the **target project's root directory** (the directory that was passed as the argument to the skill). If reviewing external code (not in a project), use the current working directory.

Determine project name from the target directory:

```bash
PROJECT_NAME=$(basename "<target-path>" | tr '[:lower:]' '[:upper:]' | tr '-' '_')
DATE=$(date +%Y-%m-%d)
FILENAME="${PROJECT_NAME}_THREAT_MODEL_${DATE}.md"
```

If the file already exists, append `_v2`, `_v3`, etc.

**Use the Write tool to create the file.** Do not just output the report to the conversation — it must be persisted as a file.

---

## Report Template

```markdown
# Threat Model: <Project Name>

**Date**: <YYYY-MM-DD>
**Analyst**: Claude (AI-assisted threat model)
**Depth**: <QUICK / STANDARD / DEEP>
**Scope**: <what was reviewed — repo path, branch, commit>

---

## Executive Summary

### Threat Overview

| Risk Level | Count |
|-----------|-------|
| CRITICAL  | <n>   |
| HIGH      | <n>   |
| MEDIUM    | <n>   |
| LOW       | <n>   |

### Overall Assessment

<2-3 sentences: overall threat posture. What's the biggest concern? What's handled well?>

### Top 3 Threats (by likelihood)

1. **<threat title>** — <one-sentence scenario + likelihood score>
2. **<threat title>** — <one-sentence scenario + likelihood score>
3. **<threat title>** — <one-sentence scenario + likelihood score>

---

## Scope & Methodology

### What Was Analyzed
- **Languages**: <detected>
- **Frameworks**: <detected>
- **Files analyzed**: <count> / <total> (<percentage>%)
- **Review depth**: <QUICK/STANDARD/DEEP>
- **Commit**: <SHA>

### Methodology
1. Reconnaissance — structural understanding, workflow discovery, state machine detection
2. Developer intent analysis — intent reconstruction, assumption mapping, gap analysis
3. Logic flaw hunting — 12-category taxonomy, adversarial workflow walkthroughs
4. Threat derivation & ranking — likelihood-based scoring, attack path mapping

### Limitations
- <what was NOT analyzed and why>
- <areas of low confidence>
- <known blind spots — e.g., no runtime analysis, no dynamic testing>
- <assumptions about deployment environment>

---

## Architecture Overview

### System Diagram
```
<ASCII diagram from Phase 1>
```

### Components
| Component | Type | Exposed | Data Handled |
|-----------|------|---------|-------------|
| <name> | <type> | <internal/external> | <data types> |

---

## Trust Boundaries

### Summary

| # | Boundary | Untrusted Side | Trusted Side | UNVALIDATED Assumptions | Risk |
|---|----------|---------------|-------------|----------------------|------|
| TB-001 | <name> | <description> | <description> | <count> | <level> |

### TB-001: <Boundary Name>

**Location**: `<file:line or architectural description>`
**Untrusted side**: <what's on this side — external users, partner APIs, etc.>
**Trusted side**: <what's on this side — internal services, database, etc.>

**Data crossing this boundary**:
<What data moves across and in which direction. Be specific — not "user data" but "user-submitted JSON body containing email, name, and role fields">

**Validation present**:
<What checks exist at this boundary — cite file:line for each>

**Validation absent**:
<What checks are MISSING — this is where threats originate. Explain WHY the gap matters, not just that it exists>

**Assumptions at this boundary**:
- A-NNN: <assumption> — <VALIDATED/UNVALIDATED/VIOLATED> — <why this matters>
- A-NNN: <assumption> — <status> — <context>

**Logic flaws at this boundary**:
- LF-NNN: <flaw> — <how it relates to this boundary>

<Repeat for each trust boundary>

---

## Developer Intent Analysis

### Alignment Overview

| Alignment | Count | Key Concern |
|-----------|-------|------------|
| FULL      | <n>   | — |
| PARTIAL   | <n>   | <summary of common gaps> |
| DIVERGENT | <n>   | <summary of critical divergences> |

### Key Intent-Reality Gaps

For each PARTIAL or DIVERGENT function — explain what the developer intended, what the code actually does, and why the gap matters.

#### G-001: <Short Title>

**Function**: `<function_name>` at `<file:line>`
**Alignment**: <PARTIAL / DIVERGENT>

**What was intended**:
<Reconstruct from docs, naming, structure — cite evidence sources>

**What actually happens**:
<What the code does line-by-line — cite specific code patterns>

**The gap**:
<Explain the divergence concretely. Not "validation is incomplete" but "the function validates email format at line 42 but never checks that the email domain is allowed, despite the comment at line 38 saying 'only corporate emails permitted'">

**Pattern**: <which common gap pattern — partial validation, happy-path-only, over-trust, etc.>
**Broken assumptions**: A-NNN, A-NNN
**Impact**: <what could go wrong because of this gap — feeds into threats>

<Repeat for each significant gap>

---

## Assumption Inventory

### Stats
- Total assumptions mapped: <n>
- VALIDATED: <n> (<percentage>%)
- UNVALIDATED: <n> (<percentage>%)
- VIOLATED: <n> (<percentage>%)

### Critical Assumptions (UNVALIDATED and VIOLATED)

For each assumption — explain what it is, where it lives, why it's unvalidated/violated, and what breaks if an attacker exploits it. A bare table row like "A-001 | user_id is valid | INPUT | UNVALIDATED" tells the reader nothing useful.

#### A-001: <Assumption description>

**Type**: <INPUT / CALLER / ENVIRONMENT / ORDERING / TRUST>
**Function**: `<function_name>` at `<file:line>`
**Status**: <UNVALIDATED / VIOLATED>

**What the code assumes**:
<Explain the assumption in context — what does the function take for granted?>

**Evidence**:
<Why is this UNVALIDATED or VIOLATED? What did you look for and not find? If VIOLATED, what code contradicts the assumption?>

**What breaks if this is false**:
<Concrete consequence — not "could lead to issues" but "attacker supplies negative quantity, charge() computes a credit instead of debit, funds transfer reverses">

**Feeds into**: T-NNN, LF-NNN

<Repeat for each UNVALIDATED/VIOLATED assumption. Group by function or workflow if there are many.>

---

## Logic Flaw Findings

### Category Summary

| Category | Flaws Found | Highest Exploitability |
|----------|------------|----------------------|
| #1 Authorization Logic Gaps | <n> | <level> |
| #2 State Machine Violations | <n> | <level> |
| #3 Race Conditions / TOCTOU | <n> | <level> |
| #4 Missing Edge Cases | <n> | <level> |
| #5 Order-of-Operations Errors | <n> | <level> |
| #6 Implicit Type Coercion | <n> | <level> |
| #7 Error Handling Logic Flaws | <n> | <level> |
| #8 Business Logic Bypass | <n> | <level> |
| #9 Trust Boundary Confusion | <n> | <level> |
| #10 Assumption Propagation | <n> | <level> |
| #11 Fail-Open vs Fail-Secure | <n> | <level> |
| #12 Temporal Logic Flaws | <n> | <level> |

### Findings

#### LF-001: <Title>

**Category**: #<N> <Category Name>
**Location**: `<file:line>`
**Exploitability**: <LOW / MEDIUM / HIGH>

**What the developer intended**:
<Reconstruct intent — what should this code path do?>

**What actually happens**:
<Describe the flaw concretely — reference specific code>

**The flaw**:
<Explain the logic error — WHY this is wrong, not just THAT it's wrong. Connect to the assumption that's broken.>

```<language>
<Quote the relevant code — the actual lines containing the flaw>
```

**How an adversary exploits this**:
<Concrete steps — from the adversarial walkthrough. Which persona found this? What do they do?>

**Related assumptions**: A-NNN — <brief context of how the assumption connects>
**Leads to threat(s)**: T-NNN

<Repeat for each logic flaw found>

---

## Threat Ranking

*Sorted by likelihood (highest first), then by impact within the same tier.*

### Confidence Scoring Framework

Every threat gets a confidence score. This is NOT about how severe the threat is — it's about how sure you are that the threat is real and accurately described. A CRITICAL threat can have LOW confidence (you think the attack is possible but haven't fully verified the path).

| Level | Meaning | Evidence Basis |
|-------|---------|---------------|
| **HIGH** | Verified through code — the flaw exists, the path is reachable, the impact is confirmed | Read the relevant code line-by-line. Traced the full call chain. Confirmed no mitigating controls exist. Verified attacker can reach the entry point. |
| **MEDIUM** | Strong indicators but incomplete verification — some part of the chain is inferred, not confirmed | Read the code but didn't trace every caller. Inferred reachability from architecture rather than verifying the exact call path. Or: confirmed the flaw but unsure about the full impact. |
| **LOW** | Plausible based on patterns but not verified in code — assessment based on structural signals, naming, or architecture rather than line-by-line reading | Identified the pattern (e.g., "no auth middleware visible") but didn't read the full function. Or: the flaw exists but the exploitability depends on runtime conditions not observable in static analysis. |

**Rule**: Never inflate confidence. If you didn't read the code, say LOW. If you read the code but not the full chain, say MEDIUM. HIGH means you traced it end-to-end.

---

### CRITICAL Risk

#### T-001: <Threat Title>

**Source**: A-NNN (<brief description>), LF-NNN (<brief description>)
**Actor**: <threat actor — from Phase 4 actor profiles>
**Boundary**: TB-NNN (<boundary name>)

**Evidence chain**:
<Trace the full path: which assumption is broken → which logic flaw it enables → how the attacker reaches it → what they achieve. This is the "why" — the reasoning that connects code analysis to the threat.>

**Scenario**:
<Concrete, step-by-step attack scenario. Not "attacker gains access" but:>
1. Attacker does X at `<endpoint/function>` (`file:line`)
2. Because assumption A-NNN is unvalidated, the system does Y
3. This triggers logic flaw LF-NNN, which allows Z
4. Attacker achieves <specific impact>

**Likelihood**: <score>
| Factor | Score | Reasoning |
|--------|-------|-----------|
| Accessibility | <1/3/5> | <why> |
| Complexity | <1/3/5> | <why> |
| Fragility | <1/3/5> | <why> |
| Motivation | <1/3/5> | <why> |
| Discoverability | <1/3/5> | <why> |

**Impact**: <CRITICAL/HIGH/MEDIUM/LOW> — <concrete impact description>

**Confidence**: <HIGH/MEDIUM/LOW>
- **What I verified**: <what code was actually read, what call chains were traced>
- **What I inferred**: <what was assumed based on structure/naming/architecture without line-by-line verification>
- **What could change this assessment**: <what additional analysis might raise or lower the confidence — e.g., "if the auth middleware is registered in a file I didn't read, this threat is invalid" or "runtime behavior of the ORM could add validation not visible in the application code">

---

### HIGH Risk

#### T-002: <Threat Title>
<same structure as CRITICAL — including full confidence breakdown>

---

### MEDIUM Risk

#### T-003: <Threat Title>
<same structure — likelihood factor table can be condensed to a single line if space is a concern, but confidence breakdown is always required>

---

### LOW Risk

#### T-004: <Threat Title>
<abbreviated — source, actor, scenario (1-2 sentences), likelihood score, impact. Confidence still required but can be a single line: "LOW — inferred from [basis], not verified in code.">

---

## Attack Paths

Multi-step chains where exploiting one threat enables the next. Each step references a threat ID so the reader can trace back to the full analysis above.

### AP-001: <Path Name>

**Actor**: <threat actor>
**Entry point**: <where the chain starts — endpoint, function, or interface>

**Steps**:
1. **T-NNN** (<threat title>): <what the attacker does> → gains <capability/access>
   - Likelihood: <score> — <why this step succeeds>
2. **T-NNN** (<threat title>): <what the attacker does next> → escalates to <level>
   - Likelihood: <score> — <why this step succeeds>
3. **T-NNN** (<threat title>): <final action> → achieves <end-state>
   - Likelihood: <score> — <why this step succeeds>

**Why this path works**: <explain the chain logic — why does step 1 enable step 2? What assumption chain breaks across the steps?>

**Cumulative likelihood**: <score>
**Final impact**: <concrete description of what the attacker ends up with>

---

## Recommendations

*Ordered by: fix the most likely threats first. Every recommendation must reference a specific threat/flaw and a specific file:line.*

### Immediate (highest-likelihood threats)

1. **Fix <threat/flaw title>** (T-NNN, LF-NNN)
   - **Where**: `<file:line>`
   - **What to change**: <specific code change — not "add validation" but "add ownership check comparing `request.user.id` against `resource.owner_id` before the update at line 42">
   - **Why this is urgent**: <connect to likelihood score — why this gets exploited first>

2. ...

### Short-term (medium-likelihood threats)

1. **Address <threat/flaw title>** (T-NNN)
   - **Where**: `<file:line>`
   - **What to change**: <specific recommendation>
   - **Why**: <connect to the assumption or flaw>

2. ...

### Long-term (architectural improvements)

1. **Strengthen <area>** (multiple threats: T-NNN, T-NNN)
   - **What**: <architectural change — e.g., "introduce centralized auth middleware" not "improve security">
   - **Why**: <which assumption chains or systemic gaps this addresses>

---

## Coverage Statement

### What Was Covered
- Workflows analyzed: <list>
- Functions analyzed: <count> at <depth> depth
- Assumption chains traced: <count>
- Adversarial walkthroughs completed: <count>

### What Was NOT Covered
- <areas skipped and why>
- <functions not analyzed at chosen depth>

### Confidence Summary

Aggregate view of how confident the analysis is across all threats. This table lets the reader quickly see which findings are well-grounded and which need further verification.

| Threat | Risk | Confidence | Basis | What Would Change It |
|--------|------|-----------|-------|---------------------|
| T-001 | CRITICAL | <HIGH/MED/LOW> | <1-sentence: what evidence supports this — e.g., "read full call chain from route handler to DB query, no auth check found"> | <what could invalidate this — e.g., "auth middleware in an unreviewed config file"> |
| T-002 | HIGH | <level> | <basis> | <what could change it> |
| ... | | | | |

**Overall confidence by area**:
- **HIGH confidence**: <areas where code was read line-by-line, full chains traced>
- **MEDIUM confidence**: <areas where key functions were read but not all callers/paths>
- **LOW confidence**: <areas assessed from architecture/structure only, not code-level>

---

## Appendices

### A. Full Assumption Inventory

Complete inventory of all assumptions mapped during Phase 2, including VALIDATED ones. This serves as the audit trail — every assumption is accounted for.

| # | Assumption | Type | Function | File:Line | Status | Validated By | Feeds Into |
|---|-----------|------|----------|-----------|--------|-------------|-----------|
| A-001 | <description> | <type> | <function> | <file:line> | <status> | <what code validates it, or "nothing"> | <T-NNN or "—"> |

### B. Adversarial Walkthrough Log

Full record of the Phase 3 adversarial persona walkthroughs. Preserves what each persona tried, what worked, and what was checked and found clear.

#### Workflow: <name> as The Scoundrel

**Goal**: <what this persona was trying to achieve>
**Entry point**: <where they started>

**Steps taken**:
1. <action> → <system response> → <exploitation attempt> → <result: found LF-NNN / checked, clear>
2. ...

**Flaws found**: LF-NNN, LF-NNN
**Checked and clear**: <list of flaw categories checked but not exploitable here, with brief reason>

<Repeat for each workflow × persona combination>

### C. Methodology Reference
- Phase 1: Reconnaissance — structural understanding, workflow + state machine discovery
- Phase 2: Developer intent analysis — intent reconstruction, assumption mapping, gap analysis
- Phase 3: Logic flaw hunting — 12-category taxonomy, adversarial personas (Scoundrel, Confused Developer, Opportunist)
- Phase 4: Threat derivation — likelihood-based ranking (5-factor model), attack path mapping
- Likelihood formula: `(Accessibility×2 + Complexity×2 + Fragility×1.5 + Motivation×1 + Discoverability×1) / 7.5`
- Risk matrix: likelihood tier × impact tier
```

---

## Report Writing Rules

1. **Every threat traces to evidence.** No threat exists without an assumption ID (A-NNN) or logic flaw ID (LF-NNN) backing it.
2. **Preserve the reasoning chain.** Tables summarize; prose sections explain WHY. Every finding needs both — a summary row for navigation AND a detailed section explaining the evidence, the developer's intent, the gap, and the consequence. A reader should never have to ask "but why is this a threat?"
3. **Likelihood first.** Sort by likelihood, not severity. Guide the reader to what's most likely to happen.
4. **Concrete scenarios.** "Authenticated user sends request X to endpoint Y, bypassing check Z, gaining access to W" — not "could lead to unauthorized access."
5. **Quote the code.** Include the relevant code snippet for every logic flaw and every violated/unvalidated assumption. The reader shouldn't need to open the codebase to understand the finding.
6. **Honest confidence.** If analysis was partial, say so. `[MEDIUM confidence]` is valid and expected.
7. **No severity inflation.** A missing edge case on a low-value field is LOW, not HIGH.
8. **No severity deflation.** An UNVALIDATED authorization assumption on a sensitive endpoint is HIGH even if exploitation is complex.
9. **Actionable recommendations.** "Add authorization check at `service.py:42` verifying `user.id == resource.owner_id`" — not "add better access control."
10. **Justify every confidence score.** HIGH/MEDIUM/LOW is meaningless without "because I read X / inferred Y / didn't verify Z." The reader needs to know what to trust and what to re-check.
11. **Coverage honesty.** State exactly what was and wasn't analyzed.

## File Writing

Write the report to the project's root directory (or the current working directory if reviewing external code):

```bash
PROJECT_NAME=$(basename "$(pwd)" | tr '[:lower:]' '[:upper:]' | tr '-' '_')
DATE=$(date +%Y-%m-%d)
FILENAME="${PROJECT_NAME}_THREAT_MODEL_${DATE}.md"
```

If the file already exists (re-analysis), append `_v2`, `_v3`, etc.

**After writing the file**, output a completion summary to the user:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Threat Model Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Report: <file path>

Threats: X CRITICAL, Y HIGH, Z MEDIUM, W LOW

Top 3 (by likelihood):
1. <threat> — <likelihood score>
2. <threat> — <likelihood score>
3. <threat> — <likelihood score>

Top recommendation: <most important fix>
```
