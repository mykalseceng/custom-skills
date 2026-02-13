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

| # | Boundary | Untrusted Side | Trusted Side | UNVALIDATED Assumptions | Risk |
|---|----------|---------------|-------------|----------------------|------|
| TB-001 | <name> | <description> | <description> | <count> | <level> |

---

## Developer Intent Summary

### Alignment Overview
| Alignment | Count | Key Concern |
|-----------|-------|------------|
| FULL      | <n>   | — |
| PARTIAL   | <n>   | <summary of common gaps> |
| DIVERGENT | <n>   | <summary of critical divergences> |

### Key Intent-Reality Gaps
| # | Function | Gap | Pattern | Impact |
|---|----------|-----|---------|--------|
| G-001 | <function> at <file:line> | <description> | <pattern name> | <potential impact> |

---

## Assumption Inventory

### Summary Stats
- Total assumptions mapped: <n>
- VALIDATED: <n> (<percentage>%)
- UNVALIDATED: <n> (<percentage>%)
- VIOLATED: <n> (<percentage>%)

### Key Assumptions (UNVALIDATED and VIOLATED only)

| # | Assumption | Type | Function | Status | Threat(s) |
|---|-----------|------|----------|--------|----------|
| A-001 | <description> | <INPUT/CALLER/ENVIRONMENT/ORDERING/TRUST> | <function> at <file:line> | UNVALIDATED | T-001 |
| A-002 | <description> | <type> | <function> at <file:line> | VIOLATED | T-003 |

---

## Logic Flaw Findings

### By Category
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

### Key Findings

#### LF-001: <Title>

**Category**: #<N> <Category Name>
**Location**: `<file:line>`
**Exploitability**: <LOW / MEDIUM / HIGH>

**Description**:
<What the logic flaw is. Reference the specific code.>

**Related assumptions**: <A-NNN, A-NNN>
**Leads to threat(s)**: <T-NNN>

---

## Threat Ranking

*Sorted by likelihood (highest first), then by impact within the same tier.*

### CRITICAL Risk

#### T-001: <Threat Title>

**Source**: <A-NNN, LF-NNN>
**Actor**: <threat actor>
**Boundary**: <trust boundary>

**Scenario**:
<2-4 sentences: what the attacker does and what happens>

**Likelihood**: <score> (<factors breakdown>)
**Impact**: <CRITICAL/HIGH/MEDIUM/LOW>
**Confidence**: <HIGH/MEDIUM/LOW>

---

### HIGH Risk

#### T-002: <Threat Title>
<same structure>

---

### MEDIUM Risk

#### T-003: <Threat Title>
<same structure>

---

### LOW Risk

#### T-004: <Threat Title>
<same structure — abbreviated, no full scenario needed>

---

## Attack Paths

### AP-001: <Path Name>

**Actor**: <threat actor>
**Steps**:
1. **T-NNN**: <action> → gains <capability>
2. **T-NNN**: <action> → escalates to <level>
3. **T-NNN**: <action> → achieves <final impact>

**Cumulative likelihood**: <score>
**Final impact**: <description>

---

## Recommendations

*Ordered by: fix the most likely threats first.*

### Immediate (highest-likelihood threats)
1. **Fix <threat/flaw>** — <specific recommendation referencing file:line>
2. ...

### Short-term (medium-likelihood threats)
1. **Address <threat/flaw>** — <recommendation>
2. ...

### Long-term (architectural improvements)
1. **Strengthen <area>** — <recommendation>
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

### Confidence Assessment
- **HIGH confidence**: <areas with thorough analysis>
- **MEDIUM confidence**: <areas with partial analysis>
- **LOW confidence**: <areas with minimal or no analysis>

---

## Appendices

### A. Full Assumption Inventory
<complete table of all assumptions, including VALIDATED>

### B. Adversarial Walkthrough Log
<summary of all persona walkthroughs from Phase 3>

### C. Methodology Reference
- Phase 1: Reconnaissance — structural understanding, workflow + state machine discovery
- Phase 2: Developer intent analysis — intent reconstruction, assumption mapping, gap analysis
- Phase 3: Logic flaw hunting — 12-category taxonomy, adversarial personas
- Phase 4: Threat derivation — likelihood-based ranking, attack path mapping
```

---

## Report Writing Rules

1. **Every threat traces to evidence.** No threat exists without an assumption ID (A-NNN) or logic flaw ID (LF-NNN) backing it.
2. **Likelihood first.** Sort by likelihood, not severity. Guide the reader to what's most likely to happen.
3. **Concrete scenarios.** "Authenticated user sends request X to endpoint Y, bypassing check Z, gaining access to W" — not "could lead to unauthorized access."
4. **Honest confidence.** If analysis was partial, say so. `[MEDIUM confidence]` is valid and expected.
5. **No severity inflation.** A missing edge case on a low-value field is LOW, not HIGH.
6. **No severity deflation.** An UNVALIDATED authorization assumption on a sensitive endpoint is HIGH even if exploitation is complex.
7. **Actionable recommendations.** "Add authorization check at `service.py:42` verifying `user.id == resource.owner_id`" — not "add better access control."
8. **Coverage honesty.** State exactly what was and wasn't analyzed.

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
