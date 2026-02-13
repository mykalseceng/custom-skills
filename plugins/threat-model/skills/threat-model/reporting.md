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

## Report Structure

The report uses a **finding-centric** structure. Each finding tells its complete story once — intent, assumptions, logic flaw, threat, and recommendation in a single place. This eliminates cross-section repetition where the same code and behavior would otherwise be described 4-8 times across separate Developer Intent, Assumption, Logic Flaw, and Threat sections.

**ID scheme**:
- **F-NNN**: Finding (the primary unit — replaces separate T-NNN, LF-NNN, G-NNN IDs)
- **A-NNN**: Assumption (referenced from findings, full inventory in Appendix A)
- **TB-NNN**: Trust boundary (architectural context)
- **CF-NNN**: Compound finding (multiple findings combining for greater impact)
- **AP-NNN**: Attack path (multi-step chains across findings)

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

### Top 3 Findings (by likelihood)

1. **<finding title>** — <one-sentence scenario + likelihood score>
2. **<finding title>** — <one-sentence scenario + likelihood score>
3. **<finding title>** — <one-sentence scenario + likelihood score>

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
<ASCII diagram from Phase 1 — use text block, not fenced code>

### Components
| Component | Type | Exposed | Data Handled |
|-----------|------|---------|-------------|
| <name> | <type> | <internal/external> | <data types> |

---

## Trust Boundaries

Architectural context — describes WHERE boundaries exist, what data crosses them, and what validation is present or absent. Findings reference these boundaries by ID.

### Summary

| # | Boundary | Untrusted Side | Trusted Side | Findings | Risk |
|---|----------|---------------|-------------|----------|------|
| TB-001 | <name> | <description> | <description> | F-NNN, F-NNN | <level> |

### TB-001: <Boundary Name>

**Location**: `<file:line or architectural description>`
**Untrusted side**: <what's on this side — external users, partner APIs, etc.>
**Trusted side**: <what's on this side — internal services, database, etc.>

**Data crossing this boundary**:
<What data moves across and in which direction. Be specific — not "user data" but "user-submitted JSON body containing email, name, and role fields">

**Validation present**:
<What checks exist at this boundary — cite file:line for each>

**Validation absent**:
<What checks are MISSING — this is where findings originate. Explain WHY the gap matters, not just that it exists>

**Related findings**: F-NNN, F-NNN

<Repeat for each trust boundary>

---

## Findings

Each finding is self-contained: intent vs reality, broken assumptions, logic flaw classification, threat assessment, and recommendation — all in one place. Code is described ONCE per finding.

Sorted by likelihood (highest first), then by impact within the same tier.

### Confidence Scoring

| Level | Meaning | Evidence Basis |
|-------|---------|---------------|
| **HIGH** | Verified through code — flaw exists, path reachable, impact confirmed | Read code line-by-line. Traced full call chain. Confirmed no mitigating controls. |
| **MEDIUM** | Strong indicators, incomplete verification | Read code but didn't trace every caller. Inferred reachability from architecture. |
| **LOW** | Plausible based on patterns, not verified in code | Identified pattern but didn't read full function. Exploitability depends on runtime. |

**Rule**: Never inflate confidence. If you didn't read the code, say LOW. If you read the code but not the full chain, say MEDIUM. HIGH means you traced it end-to-end.

---

### CRITICAL Risk

#### F-001: <Finding Title>

**Risk**: CRITICAL | **Likelihood**: <score> | **Confidence**: <HIGH/MEDIUM/LOW>
**Location**: `<file:line>`
**Boundary**: TB-NNN
**Actor**: <threat actor>
**Logic Flaw Category**: #<N> <Category Name>

**Intent vs Reality**:

*What was intended*:
<Reconstruct from docs, naming, structure — cite evidence>

*What actually happens*:
<What the code does — cite specific code patterns>

```<language>
<The actual lines containing the flaw — quote the code ONCE here>
```

*The gap*:
<Explain the divergence concretely — WHY this is wrong, not just THAT it's wrong>

**Assumptions Broken**:
- A-NNN: <assumption> — <VIOLATED/UNVALIDATED> — <what the code assumes + evidence it's broken + what breaks if false>
- A-NNN: <assumption> — <status> — <same structure>

**Attack Scenario**:
1. Attacker does X at `<endpoint/function>` (`file:line`)
2. Because assumption A-NNN is broken, the system does Y
3. This triggers the logic flaw, which allows Z
4. Attacker achieves <specific impact>

**Likelihood**:
| Factor | Score | Reasoning |
|--------|-------|-----------|
| Accessibility | <1-5> | <why> |
| Complexity | <1-5> | <why> |
| Fragility | <1-5> | <why> |
| Motivation | <1-5> | <why> |
| Discoverability | <1-5> | <why> |

**Impact**: <CRITICAL/HIGH/MEDIUM/LOW> — <concrete impact description>

**Confidence Breakdown**:
- **What I verified**: <what code was read, what chains traced>
- **What I inferred**: <what was assumed without line-by-line verification>
- **What could change this assessment**: <what might raise or lower confidence>

**Recommendation**:
- **Where**: `<file:line>`
- **What to change**: <specific code change — not "add validation" but "add ownership check comparing `request.user.id` against `resource.owner_id` before the update at line 42">
- **Why this is urgent**: <connect to likelihood — why this gets exploited first>

---

<Repeat for each CRITICAL finding>

### HIGH Risk

#### F-002: <Finding Title>
<Same structure as CRITICAL — full confidence breakdown required>

---

### MEDIUM Risk

#### F-003: <Finding Title>
<Same structure — likelihood factor table can be condensed to a single line if space is a concern, but confidence breakdown is always required>

---

### LOW Risk

#### F-004: <Finding Title>
<Abbreviated — location, intent vs reality (condensed), assumptions broken (list), scenario (1-2 sentences), likelihood score, impact, confidence (single line: "LOW — inferred from [basis], not verified in code"). Recommendation still required.>

---

### Compound Findings

When multiple findings combine to create an impact greater than any individual finding.

#### CF-001: <Compound Title>

**Component findings**: F-NNN, F-NNN, F-NNN

**Combined effect**: <explain why the combination is worse than individual findings — what chain breaks across the findings?>

**Attack scenario**: <end-to-end scenario using the compound chain>

---

## Attack Paths

Multi-step chains where exploiting one finding enables the next. Each step references a finding ID so the reader can trace back to the full analysis.

### AP-001: <Path Name>

**Actor**: <threat actor>
**Entry point**: <where the chain starts — endpoint, function, or interface>

**Steps**:
1. **F-NNN** (<finding title>): <what the attacker does> → gains <capability/access>
   - Likelihood: <score> — <why this step succeeds>
2. **F-NNN** (<finding title>): <what the attacker does next> → escalates to <level>
   - Likelihood: <score> — <why this step succeeds>
3. **F-NNN** (<finding title>): <final action> → achieves <end-state>
   - Likelihood: <score> — <why this step succeeds>

**Why this path works**: <explain the chain logic — why does step 1 enable step 2? What assumption chain breaks across the steps?>

**Cumulative likelihood**: <score>
**Final impact**: <concrete description of what the attacker ends up with>

---

## Recommendations Summary

Condensed priority-ordered list for teams that need action items. Full detail is in each finding's Recommendation section.

### Immediate (highest-likelihood findings)

1. **<fix title>** (F-NNN) — `<file:line>` — <one-line change description>
2. ...

### Short-term (medium-likelihood findings)

1. **<fix title>** (F-NNN) — `<file:line>` — <one-line change description>
2. ...

### Long-term (architectural improvements)

1. **<fix title>** (F-NNN, F-NNN) — <architectural change description>
2. ...

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

| Finding | Risk | Confidence | Basis | What Would Change It |
|---------|------|-----------|-------|---------------------|
| F-001 | CRITICAL | <HIGH/MED/LOW> | <1-sentence evidence basis> | <what could invalidate> |
| F-002 | HIGH | <level> | <basis> | <what could change it> |
| ... | | | | |

**Overall confidence by area**:
- **HIGH confidence**: <areas where code was read line-by-line, full chains traced>
- **MEDIUM confidence**: <areas where key functions were read but not all callers/paths>
- **LOW confidence**: <areas assessed from architecture/structure only, not code-level>

---

## Appendices

### A. Full Assumption Inventory

**Required.** Complete inventory of ALL assumptions mapped during Phase 2, including VALIDATED ones. This is the audit trail — every assumption accounted for. Even validated assumptions matter: they document what was checked and found sound, so future reviewers know what's already been verified.

| # | Assumption | Type | Function | File:Line | Status | Validated By | Feeds Into |
|---|-----------|------|----------|-----------|--------|-------------|-----------|
| A-001 | <description> | <INPUT/CALLER/ENVIRONMENT/ORDERING/TRUST> | <function> | <file:line> | <VALIDATED/UNVALIDATED/VIOLATED> | <what code validates it, or "nothing"> | <F-NNN or "—"> |

### B. Adversarial Walkthrough Log

**Required.** Record of Phase 3 adversarial persona walkthroughs. This log is critical for demonstrating coverage — it shows not just what was found, but what was tested and ruled out. A threat model without a walkthrough log has no evidence of adversarial testing.

#### Workflow: <name> as The Scoundrel

**Goal**: <what this persona was trying to achieve>
**Entry point**: <where they started>

**Steps taken**:
1. <action> → <system response> → <exploitation attempt> → <result: found F-NNN / checked, clear>
2. ...

**Findings**: F-NNN, F-NNN
**Checked and clear**: <list of flaw categories checked but not exploitable here, with brief reason>

<Repeat for each workflow x persona combination>

### C. Logic Flaw Category Summary

Aggregate view of which logic flaw categories were found across all findings.

| Category | Findings | Highest Risk |
|----------|---------|-------------|
| #1 Authorization Logic Gaps | F-NNN, F-NNN | <level> |
| #2 State Machine Violations | F-NNN | <level> |
| #3 Race Conditions / TOCTOU | — | — |
| #4 Missing Edge Cases | — | — |
| #5 Order-of-Operations Errors | — | — |
| #6 Implicit Type Coercion | — | — |
| #7 Error Handling Logic Flaws | — | — |
| #8 Business Logic Bypass | F-NNN | <level> |
| #9 Trust Boundary Confusion | F-NNN | <level> |
| #10 Assumption Propagation | — | — |
| #11 Fail-Open vs Fail-Secure | — | — |
| #12 Temporal Logic Flaws | — | — |

### D. Methodology Reference
- Phase 1: Reconnaissance — structural understanding, workflow + state machine discovery
- Phase 2: Developer intent analysis — intent reconstruction, assumption mapping, gap analysis
- Phase 3: Logic flaw hunting — 12-category taxonomy, adversarial personas (Scoundrel, Confused Developer, Opportunist)
- Phase 4: Threat derivation — likelihood-based ranking (5-factor model), attack path mapping
- Likelihood formula: `(Accessibility*2 + Complexity*2 + Fragility*1.5 + Motivation*1 + Discoverability*1) / 7.5`
- Risk matrix: likelihood tier x impact tier

              | LOW Impact  | MED Impact  | HIGH Impact | CRIT Impact |
--------------+-------------+-------------+-------------+-------------+
HIGH Likely   | MEDIUM      | HIGH        | CRITICAL    | CRITICAL    |
MED Likely    | LOW         | MEDIUM      | HIGH        | CRITICAL    |
LOW Likely    | LOW         | LOW         | MEDIUM      | HIGH        |
```

---

## Report Writing Rules

1. **Every finding traces to evidence.** No finding exists without assumption IDs (A-NNN) backing it. Code snippets required for every finding.
2. **Describe code once.** Each finding's "Intent vs Reality" section is the single source of truth for that code. Trust Boundaries describe where boundaries exist; Findings describe what's wrong at those boundaries. Never duplicate code snippets across sections.
3. **Preserve the reasoning chain.** Each finding must show: intent → reality → gap → broken assumptions → attack scenario → likelihood → impact → recommendation. A reader should never have to jump across sections to understand a finding.
4. **Likelihood first.** Sort by likelihood, not severity. Guide the reader to what's most likely to happen.
5. **Concrete scenarios.** "Authenticated user sends request X to endpoint Y, bypassing check Z, gaining access to W" — not "could lead to unauthorized access."
6. **Quote the code.** Include the relevant code snippet in every finding. The reader shouldn't need to open the codebase.
7. **Honest confidence.** If analysis was partial, say so. MEDIUM confidence is valid and expected.
8. **No severity inflation.** A missing edge case on a low-value field is LOW, not HIGH.
9. **No severity deflation.** An UNVALIDATED authorization assumption on a sensitive endpoint is HIGH even if exploitation is complex.
10. **Actionable recommendations.** "Add authorization check at `service.py:42` verifying `user.id == resource.owner_id`" — not "add better access control."
11. **Justify every confidence score.** HIGH/MEDIUM/LOW is meaningless without "because I read X / inferred Y / didn't verify Z."
12. **Coverage honesty.** State exactly what was and wasn't analyzed.
13. **Appendices A and B are required.** The Assumption Inventory and Adversarial Walkthrough Log are the evidence trail that backs the findings. A report without them is unverifiable.

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

Findings: X CRITICAL, Y HIGH, Z MEDIUM, W LOW

Top 3 (by likelihood):
1. <finding> — <likelihood score>
2. <finding> — <likelihood score>
3. <finding> — <likelihood score>

Top recommendation: <most important fix>
```
