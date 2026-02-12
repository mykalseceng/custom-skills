# Diff Mode: PR / Commit Security Review

Security-focused review of code changes — PRs, commits, or diff files. Combines git history analysis, blast radius calculation, and adversarial thinking.

Use this file when the review target is a PR URL, commit SHA, or diff file — NOT a full repository.

---

## 1. Intake & Triage

### Extract Changes

```bash
# From commit range
git diff --stat <baseline>..<target>
git diff --name-only <baseline>..<target>
git diff <baseline>..<target>

# From single commit
git show --stat <sha>
git diff <sha>^..<sha>

# From PR (GitHub CLI)
gh pr diff <number>
gh pr view <number> --json files,additions,deletions

# From diff file
cat <path-to-diff>
```

### Classify Codebase Size

| Size | Files Changed | Lines Changed | Approach |
|------|--------------|---------------|----------|
| **Small** | 1-5 | <100 | Review every line |
| **Medium** | 6-20 | 100-500 | Prioritize by risk score |
| **Large** | 21-50 | 500-2000 | Focus on HIGH risk files, spot-check others |
| **Very Large** | 50+ | 2000+ | Risk-score everything, deep-review top 10-15 files |

### Risk-Score Each File

Score each changed file 1-5 based on:

| Signal | +1 | +2 |
|--------|-----|-----|
| Handles auth/authz | Yes | Central auth module |
| Processes user input | Indirectly | Direct from request |
| Touches crypto/secrets | Uses crypto lib | Implements crypto logic |
| Database operations | Read queries | Write/migration changes |
| Infrastructure/deploy | Config change | Permission/network change |
| External API calls | Outbound calls | Inbound webhook handlers |
| File/path operations | Read operations | Write/delete with user paths |

Sort files by risk score descending. This determines review order.

## 2. Context Building

For each HIGH risk file (score 4-5), and for any function that was modified:

1. **Read the baseline version** of the changed functions (pre-change)
2. **Read the modified version** (post-change)
3. **Apply [deep-context.md](deep-context.md) methodology** to both versions of changed functions
4. **Read callers** of modified functions to understand blast radius

For MEDIUM risk files (score 2-3):
- Read the diff hunks with surrounding context (use `git diff -U10`)
- Understand what the change does, but skip full micro-analysis

For LOW risk files (score 1):
- Scan the diff for obvious issues (secrets, hardcoded creds, debug code)

## 3. Changed Code Analysis

For each diff hunk in HIGH/MEDIUM risk files:

### BEFORE / AFTER Template

```markdown
### File: `<path>` (lines <range>)

**BEFORE** (baseline behavior):
[What this code did before the change — from context building]

**AFTER** (new behavior):
[What this code does after the change]

**Behavioral Change**:
[What actually changed in behavior — not just syntax]

**Security Implications**:
- [Does this add/remove/change a security check?]
- [Does this change input validation?]
- [Does this change data flow or trust boundaries?]
- [Does this change error handling in a security-relevant way?]

**Risk Assessment**: [LOW / MEDIUM / HIGH / CRITICAL]
**Reason**: [Why this risk level]
```

### Red Flags — Immediate Escalation Triggers

These patterns **override risk scoring**. If ANY of these appear in a diff — even in a file scored LOW — immediately escalate to HIGH and apply full adversarial analysis (Section 7). Do not wait for blast radius or test coverage assessment. Investigate now.

#### Tier 1: Stop Everything (automatic CRITICAL until disproven)

| Red Flag | Why It's Critical | Detection |
|----------|------------------|-----------|
| Removed code from commits containing "security", "CVE", "fix", "vuln", "patch" | Undoing a security fix is a regression | `git log -p --all -S "<removed-line>" -- <file>` then check commit messages |
| Access control modifiers removed (`onlyOwner`, `internal→external`, `private→public`, auth middleware removed from route) | Direct path to privilege escalation | Compare route/handler definitions BEFORE vs AFTER |
| Validation removed without replacement | Opens input to attack | Diff shows removed `validate`, `sanitize`, `check`, `assert`, `guard` calls with no equivalent added |
| Hardcoded secrets added (API keys, passwords, tokens in source) | Immediate credential exposure | Grep diff for patterns: `key=`, `secret=`, `password=`, `token=`, `AKIA`, `BEGIN.*PRIVATE KEY` |
| Deserialization of untrusted data added (`pickle.load`, `yaml.load`, `unserialize`, `ObjectInputStream`, `eval`) | Direct RCE vector | Grep diff for deserialization functions |

#### Tier 2: Investigate Immediately (automatic HIGH until assessed)

| Red Flag | Why It's Dangerous | Detection |
|----------|-------------------|-----------|
| External calls added without error handling or input validation | SSRF, injection, or data leak | New `requests.get`, `fetch`, `http.Get`, `axios` calls in diff |
| Crypto algorithm changed (especially downgrade: SHA256→MD5, AES→DES, bcrypt→SHA1) | Weakens security guarantees | Diff shows changed crypto function calls or constants |
| Error handling weakened (specific catches → catch-all, error detail added to response) | Information disclosure, silent failures | Diff shows broader exception handlers or error messages with stack traces |
| New user input path without sanitization | Injection vector | New route/endpoint/handler that reads `request.params`, `req.body`, `args`, etc. without validation |
| Permission or role logic changed | Privilege escalation, broken access control | Diff modifies role checks, permission maps, ACL definitions |
| Database migration removes constraints (NOT NULL, UNIQUE, CHECK, FOREIGN KEY) | Data integrity loss enables attacks | Diff shows `DROP CONSTRAINT`, `ALTER TABLE ... DROP`, removed validations in migration files |

#### Tier 3: Flag for Review (elevate risk score by +2)

| Red Flag | Why It Matters |
|----------|---------------|
| New dependencies added (especially native/binary) | Supply chain risk |
| Config changes that relax security (CORS widened, CSP loosened, debug enabled) | Attack surface expansion |
| File path construction with user input | Path traversal |
| Serialization/deserialization format changed | Data integrity, potential RCE |
| TODO/FIXME with security context removed without resolution | Lost security intent |
| Commit message contains "temporary", "workaround", "quick fix", "disable", "skip", "bypass" | Technical debt with security implications |

### Escalation Flow

```
Diff hunk → Check against red flag tiers
  ├─ Tier 1 match → CRITICAL. Full adversarial analysis (Section 7). Immediately.
  ├─ Tier 2 match → HIGH. Promote file to full deep-context + adversarial analysis.
  ├─ Tier 3 match → +2 risk score. Re-evaluate file priority.
  └─ No match → Continue with normal risk-scored review.
```

### Other Patterns to Watch

Beyond the escalation triggers, also watch for:
- Error handling changed to be less strict (catch-all, silent failures)
- Direct SQL/command string construction
- Inconsistency with how similar code is handled elsewhere in the codebase

## 4. Git History Analysis

### Blame Analysis

For each removed or modified line in HIGH risk files:

```bash
# Who wrote the original code and when
git blame <baseline> -- <file> | grep -n "relevant function"

# History of the specific file
git log --oneline -20 -- <file>

# Was this code added for a security fix?
git log --all --oneline --grep="security\|fix\|vuln\|CVE\|XSS\|inject\|auth\|bypass" -- <file>
```

### Regression Detection

Check if any of these dangerous patterns appear:
- **Code removed for security, now re-added** — search commit history for security-related removals
- **Security fix reverted** — compare current diff against previous security commits
- **TODO/FIXME security items removed without resolution**

```bash
# Check for security-related commits on this file
git log --all --oneline --grep="security\|vuln\|fix\|patch" -- <changed-files>

# Check if removed lines were added in a security context
git log -p --all -S "<removed-line-content>" -- <file>
```

### Commit Message Red Flags

- "temporary fix" / "quick fix" / "workaround" — may introduce technical debt with security implications
- "disable" / "skip" / "bypass" — may remove security controls
- "revert" — may undo security fixes

## 5. Test Coverage Analysis

### Find Tests for Changed Code

```bash
# Search for test files related to changed files
# Node/JS
find . -name "*.test.*" -o -name "*.spec.*" | xargs grep -l "<changed-function-name>"

# Python
find . -name "test_*" -o -name "*_test.py" | xargs grep -l "<changed-function-name>"

# Go
find . -name "*_test.go" | xargs grep -l "<changed-function-name>"

# Java
find . -name "*Test.java" -o -name "*Tests.java" | xargs grep -l "<changed-function-name>"
```

### Test Gap Assessment

| Change | Has Tests? | Test Quality | Risk Elevation |
|--------|-----------|-------------|---------------|
| New function with user input | NO | — | +2 risk levels |
| Modified auth logic | YES | Doesn't test edge cases | +1 risk level |
| New endpoint | NO | — | +2 risk levels |
| Modified validation | YES | Tests happy path only | +1 risk level |

Rule: **New security-relevant code without tests = automatic risk elevation**.

## 6. Blast Radius Assessment

For each modified function, count how many other functions call it:

```bash
# Find callers of modified function
grep -rn "<function-name>" --include="*.{py,js,ts,go,java,rb,php}" | grep -v "def \|function \|func \|test\|spec\|mock"
```

### Blast Radius Classification

| Callers | Blast Radius | Review Implication |
|---------|-------------|-------------------|
| 1-5 | LOW | Review the callers for context |
| 6-20 | MEDIUM | Verify callers handle the behavioral change correctly |
| 21-50 | HIGH | This change affects a significant portion of the codebase |
| 50+ | CRITICAL | This is a foundational change — full regression risk analysis needed |

### Priority Matrix

```
              │ LOW Blast  │ MED Blast  │ HIGH Blast │ CRIT Blast │
──────────────┼────────────┼────────────┼────────────┼────────────┤
HIGH Risk     │ HIGH       │ CRITICAL   │ CRITICAL   │ CRITICAL   │
MED Risk      │ MEDIUM     │ HIGH       │ HIGH       │ CRITICAL   │
LOW Risk      │ LOW        │ MEDIUM     │ MEDIUM     │ HIGH       │
```

## 7. Adversarial Analysis

For every file/change rated HIGH risk or above after blast radius assessment:

### Exploit Scenario Template (Diff-Specific)

```markdown
### Change: [file:line range]

**What changed**: [behavioral change in one sentence]

**Attacker Model**:
- Who: [unauth / auth user / admin / compromised service]
- Access: [what they can reach]
- Interface: [which endpoint/input]

**Exploit scenario**:
1. Before this change, [what prevented the attack]
2. After this change, attacker can [specific action]
3. By sending [specific input/request] to [endpoint]
4. This results in [specific impact]

**Exploitability**: [EASY / MEDIUM / HARD]

**PoC**:
```
[Exact request or code that exploits the change]
```

**Recommendation**: [specific code fix]
```

## 8. Cross-Cutting Pattern Detection

Check if the diff breaks consistency with the rest of the codebase:

```bash
# Find how similar patterns are handled elsewhere
# Example: if the diff changes input validation for one endpoint,
# check how other endpoints validate the same input type
grep -rn "<validation-pattern>" --include="*.{py,js,ts,go,java,rb,php}" | grep -v "<changed-file>"
```

### Consistency Checks

- If the diff adds a new endpoint, does it follow the auth pattern of other endpoints?
- If the diff changes error handling, is it consistent with the project's error handling pattern?
- If the diff adds new validation, does it match validation patterns elsewhere?
- If the diff introduces a new dependency, is it consistent with the project's dependency management?

Flag any inconsistencies as findings — they indicate either a bug in the diff or tech debt in the existing code.

## Diff Mode Output

Feed all findings into [reporting.md](reporting.md) using the **Diff Mode Report** template. Include:
- File risk scores
- BEFORE/AFTER analysis for HIGH risk changes
- Git history findings
- Test coverage gaps
- Blast radius assessment
- Exploit scenarios for HIGH/CRITICAL findings
- Cross-cutting pattern inconsistencies
- Final verdict: APPROVE / REJECT / CONDITIONAL APPROVE
