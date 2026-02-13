# Report Generation

Templates and guidelines for producing the final security review report. Two templates: one for Repo Mode (full codebase) and one for Diff Mode (PR/commit).

---

## Formatting Guidelines

### Severity Indicators

Use these consistently throughout the report:

- `[CRITICAL]` — Immediate exploitation risk, highest priority
- `[HIGH]` — Serious vulnerability, requires prompt remediation
- `[MEDIUM]` — Moderate risk, should be addressed in normal development cycle
- `[LOW]` — Minor issue, address when convenient

### Code References

Always use `file:line` format for references:
- Single line: `src/auth/login.ts:42`
- Range: `src/auth/login.ts:42-58`
- Multiple locations: `src/auth/login.ts:42`, `src/auth/session.ts:15`

### Code Blocks

Always specify the language for syntax highlighting:
````markdown
```python
# vulnerable code
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")

# recommended fix
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```
````

### Finding IDs

Number findings sequentially: `F-001`, `F-002`, etc. This enables cross-referencing.

---

## Repo Mode Report Template

File name: `<PROJECT>_SECURITY_REVIEW_<YYYY-MM-DD>.md`

```markdown
# Security Review: <Project Name>

**Date**: <YYYY-MM-DD>
**Reviewer**: Claude (AI-assisted security review)
**Depth**: <QUICK / STANDARD / DEEP>
**Scope**: <what was reviewed — repo path, branch, commit>

---

## Executive Summary

### Severity Overview

| Severity | Count |
|----------|-------|
| CRITICAL | <n> |
| HIGH     | <n> |
| MEDIUM   | <n> |
| LOW      | <n> |

### Overall Risk Assessment

<1-3 sentences: overall security posture of the codebase>

### Top 3 Risks

1. **<risk title>** — <one-sentence description + location>
2. **<risk title>** — <one-sentence description + location>
3. **<risk title>** — <one-sentence description + location>

---

## Scope & Methodology

### What Was Reviewed
- **Languages**: <languages detected>
- **Frameworks**: <frameworks detected>
- **Files analyzed**: <count> / <total> (<percentage>%)
- **Review depth**: <QUICK/STANDARD/DEEP>
- **Commit**: <SHA>

### Methodology
1. Reconnaissance (language detection, entry points, dependencies, secrets, auth mapping)
2. Deep code comprehension (line-by-line analysis of key functions informed by recon)
3. Threat modeling (architecture, trust boundaries, STRIDE analysis)
4. OWASP Top 10 vulnerability hunting (systematic grep + manual review)
5. Deep dive on HIGH/CRITICAL findings (exploit scenario development)

### Limitations
- <what was NOT reviewed and why>
- <areas of low confidence>
- <known blind spots — e.g., no runtime analysis, no DAST>

---

## Threat Model Summary

### Architecture
```
<ASCII diagram from threat-model.md Phase 3>
```

### Trust Boundaries
| # | Boundary | Risk | Key Concern |
|---|----------|------|-------------|
| 1 | <name> | <level> | <summary> |

### Key Attacker Profiles
- **<attacker>**: <what they can do, highest-value target>

---

## Findings

### [CRITICAL] F-001: <Title>

**OWASP Category**: <A01-A10>
**Location**: `<file:line>`
**Confidence**: <HIGH / MEDIUM / LOW>

**Description**:
<What the vulnerability is. Be specific — reference exact code.>

**Attack Scenario**:
- **Attacker**: <who>
- **Steps**:
  1. <step with specific details>
  2. <step>
  3. <step>
- **Impact**: <concrete impact>
- **Exploitability**: <EASY / MEDIUM / HARD>

**Proof of Concept**:
```
<exact request/input that triggers the vulnerability>
```

**Recommendation**:
```<language>
<specific code fix>
```

---

### [HIGH] F-002: <Title>
<same structure as above>

---

### [MEDIUM] F-003: <Title>

**OWASP Category**: <A01-A10>
**Location**: `<file:line>`
**Confidence**: <HIGH / MEDIUM / LOW>

**Description**:
<description>

**Recommendation**:
```<language>
<specific code fix>
```

---

### [LOW] F-004: <Title>
<same structure as MEDIUM — no exploit scenario needed>

---

## Attack Surface Summary

### Entry Points
| Endpoint/Handler | Method | Auth | Risk Level |
|-----------------|--------|------|-----------|
| <path> | <method> | <yes/no> | <level> |

### Auth Coverage Gaps
- <endpoint or feature lacking proper auth>

---

## Dependency Analysis

### Overview
- **Total dependencies**: <count>
- **Lock file**: <present/absent>
- **Last updated**: <date>

### Concerns
- <any flagged dependencies>

---

## Recommendations

### Immediate (address before next release)
1. <recommendation referencing F-XXX>

### Short-term (within 1-2 sprints)
1. <recommendation>

### Long-term (architectural improvements)
1. <recommendation>

---

## Appendices

### A. Files Reviewed
<list of files that were analyzed in depth>

### B. Tools & Patterns Used
<grep patterns, analysis techniques>

### C. Glossary
<any terms that need definition for the target audience>
```

---

## Diff Mode Report Template

File name: `<PROJECT>_DIFF_REVIEW_<YYYY-MM-DD>.md`

```markdown
# Diff Security Review: <Project Name>

**Date**: <YYYY-MM-DD>
**Reviewer**: Claude (AI-assisted security review)
**Scope**: <PR #/URL, commit range, or diff file>
**Baseline**: <branch/tag/SHA>

---

## Executive Summary

### Verdict: <APPROVE / REJECT / CONDITIONAL APPROVE>

<1-3 sentences explaining the verdict>

### Severity Overview

| Severity | Count |
|----------|-------|
| CRITICAL | <n> |
| HIGH     | <n> |
| MEDIUM   | <n> |
| LOW      | <n> |

---

## What Changed

### File Summary
| File | Risk Score | Lines +/- | Key Change |
|------|-----------|-----------|-----------|
| <path> | <1-5> | +<n>/-<n> | <one-line summary> |

### Stats
- Files changed: <n>
- Lines added: <n>
- Lines removed: <n>
- New files: <n>
- Deleted files: <n>

---

## Critical Findings

### [SEVERITY] F-001: <Title>

**File**: `<path:line>`
**Change Type**: <new code / modified code / removed check>

**BEFORE**:
```<language>
<code before change>
```

**AFTER**:
```<language>
<code after change>
```

**Behavioral Change**: <what actually changed>

**Historical Context**:
- Original author: <from git blame>
- Last modified: <date>
- Related commits: <any security-related commits on this file>

**Attack Scenario**:
<from adversarial analysis — who can exploit this, how, and what impact>

**Exploitability**: <EASY / MEDIUM / HARD>

**Recommendation**:
```<language>
<specific fix>
```

---

## Test Coverage Analysis

| Changed Code | Test Exists? | Test Quality | Risk Impact |
|-------------|-------------|-------------|-------------|
| <function/file> | YES/NO | <assessment> | <risk elevation> |

### Missing Tests
- <list of changes that need tests>

---

## Blast Radius Analysis

| Modified Function | Callers | Blast Radius | Impact |
|------------------|---------|-------------|--------|
| <function> | <count> | <LOW-CRITICAL> | <description> |

---

## Historical Context

### Git Blame Findings
- <relevant history — who wrote the original code, security-related changes>

### Regression Risks
- <any code that was previously removed for security now being re-added>
- <any reverted security fixes>

---

## Recommendations

### Must Fix Before Merge
1. <blocking issues>

### Should Fix Before Merge
1. <important but not blocking>

### Consider for Follow-up
1. <improvements that can be separate PRs>

---

## Analysis Methodology

- Changed files risk-scored and prioritized
- HIGH risk files analyzed with deep context building
- BEFORE/AFTER behavioral analysis per diff hunk
- Git history analyzed for security context and regressions
- Test coverage gaps identified and risk-elevated
- Blast radius calculated for all modified functions
- Adversarial analysis performed on HIGH+ risk changes
- Cross-cutting patterns checked for consistency
```

---

## Report Writing Rules

1. **Be specific, never vague.** "SQL injection in `user_service.py:42`" not "potential injection issue."
2. **Show don't tell.** Include code snippets for both the vulnerability and the fix.
3. **Concrete impact.** "Attacker can read any user's data" not "could lead to data exposure."
4. **Honest confidence.** If you're not sure, say so. `[MEDIUM confidence]` is valid.
5. **Actionable recommendations.** Code fixes, not just "add validation."
6. **No severity inflation.** A missing header is LOW, not HIGH. An informational finding is not a vulnerability.
7. **No severity deflation.** An auth bypass is CRITICAL even if it's in a "low-traffic" endpoint.
8. **Coverage statement.** Always state what was and wasn't reviewed.

## File Writing

Write the report to the project's root directory (or the current working directory if reviewing external code):

```bash
# Determine project name from directory
PROJECT_NAME=$(basename "$(pwd)" | tr '[:lower:]' '[:upper:]' | tr '-' '_')
DATE=$(date +%Y-%m-%d)

# Repo mode
FILENAME="${PROJECT_NAME}_SECURITY_REVIEW_${DATE}.md"

# Diff mode
FILENAME="${PROJECT_NAME}_DIFF_REVIEW_${DATE}.md"
```

If the file already exists (re-review), append a counter: `_v2`, `_v3`, etc.

After writing, inform the user:
- Report file path
- Summary of findings (severity counts)
- Top 3 most important findings
- Recommended next actions
