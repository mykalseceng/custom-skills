---
name: codebase-security-review
description: >
  Performs deep code comprehension followed by security review of entire codebases
  or code changes. Combines line-by-line analysis with threat modeling, OWASP
  vulnerability hunting, and exploit scenario building. Use for full repo security
  audits, PR/commit security review, or threat modeling any codebase.
  Language-agnostic (Python, Go, TypeScript, Rust, Java, C/C++, Ruby, PHP, etc.).
allowed-tools:
  - Read
  - Write
  - Grep
  - Glob
  - Bash
---

# Codebase Security Review

Deep code comprehension + security analysis. Two modes, one methodology.

## Companion Files

All methodology docs are **sibling files in the same directory as this SKILL.md** (not in a subdirectory):

- **[deep-context.md](deep-context.md)** — Phase 1: Line-by-line micro-analysis methodology
- **[reconnaissance.md](reconnaissance.md)** — Phase 2: Language, framework, entry point, dependency discovery
- **[threat-model.md](threat-model.md)** — Phase 3: Architecture, trust boundaries, STRIDE analysis
- **[vulnerability-hunting.md](vulnerability-hunting.md)** — Phases 4-5: OWASP hunting + exploit deep dive
- **[reporting.md](reporting.md)** — Phase 6: Report templates (repo + diff modes)
- **[diff-review.md](diff-review.md)** — Diff mode: PR/commit analysis methodology

## Two Modes

| Aspect | Repo Mode (Full Codebase) | Diff Mode (PR / Commit) |
|--------|--------------------------|------------------------|
| **Input** | Path to repository | PR URL, commit SHA, or .diff/.patch file |
| **Phase 1** | Deep Context (all key functions) | Deep Context (changed functions + callers) |
| **Phase 2** | Reconnaissance | Intake & Triage |
| **Phase 3** | Threat Model | Changed Code Analysis |
| **Phase 4** | OWASP Vulnerability Hunt | Git History + Blast Radius |
| **Phase 5** | Deep Dive on HIGHs | Adversarial Analysis on HIGHs |
| **Phase 6** | Report | Report |
| **Output** | `<PROJECT>_SECURITY_REVIEW_<DATE>.md` | `<PROJECT>_DIFF_REVIEW_<DATE>.md` |

## Core Principles

1. **Comprehend Before Judging** — Build a complete mental model before making any security claims. Findings without deep understanding are hallucinations.
2. **Architecture-First** — Map the system before hunting bugs. Understanding component boundaries and trust relationships prevents false positives and reveals design-level flaws.
3. **Attacker-Centric** — Every finding needs a realistic exploit scenario. "This could be bad" is not a finding. "An unauthenticated attacker sends X to endpoint Y, causing Z" is.
4. **Evidence-Based** — Every claim needs file:line references, git history, or data flow traces. No hand-waving.
5. **Language-Agnostic** — Same methodology regardless of language. Language-specific detection patterns are in the reference docs.
6. **Honest About Limits** — State coverage percentage, confidence level, what was skipped, and why. A review that says "I checked everything" checked nothing.

## Rationalizations to Reject

| Rationalization | Why It's Wrong | What to Do Instead |
|----------------|---------------|-------------------|
| "I get the gist" | Gist-level reading misses edge cases, off-by-ones, and subtle state bugs | Line-by-line analysis per [deep-context.md](deep-context.md) |
| "Jump straight to vuln hunting" | Skipping context = hallucinated findings and missed design flaws | Complete Phases 1-3 before hunting |
| "Small repo, quick review" | Small ≠ simple; a 200-line crypto lib has more risk than a 10k-line CRUD app | Classify by RISK not LOC |
| "I know this language well" | Familiarity breeds blind spots; you skip what feels obvious | Follow the methodology every time |
| "No auth/crypto = low risk" | Injection, SSRF, path traversal, deserialization are everywhere | Full OWASP check regardless |
| "Dependencies not my problem" | 80%+ of exploited vulns are in transitive dependencies | Check manifests + lock files |
| "Just a refactor, no security impact" | Refactors break invariants, change call order, remove guards | Analyze as HIGH risk until proven LOW |
| "I'll explain verbally" | No artifact = findings lost, no accountability, no re-review | Always write the report per [reporting.md](reporting.md) |

## Review Depth

| Depth | When to Use | File Analysis | Grep Patterns | Threat Model |
|-------|------------|--------------|---------------|-------------|
| **QUICK** | <20 files, low-risk, time-constrained | Entry points + high-risk functions only | Core injection + auth | Trust boundaries only |
| **STANDARD** | 20-200 files, typical review | All public/exported functions | Full OWASP set | Full STRIDE per boundary |
| **DEEP** | >200 files, high-value target, audit engagement | Every non-trivial function incl. helpers | Full OWASP + language-specific | Full STRIDE + attacker profiles |

## Risk Classification

| Severity | Trigger Criteria |
|----------|-----------------|
| **CRITICAL** | RCE, auth bypass, SQL injection with data exfil, hardcoded secrets in production, deserialization of untrusted data |
| **HIGH** | Privilege escalation, SSRF, stored XSS, path traversal to sensitive files, broken access control on sensitive endpoints |
| **MEDIUM** | Reflected XSS, CSRF, information disclosure (stack traces, verbose errors), missing rate limiting on auth endpoints |
| **LOW** | Missing security headers, overly permissive CORS (non-credentialed), debug mode in non-prod configs, minor logging gaps |

## Workflow: Repo Mode

```
┌─────────────────────────────────────────────────────────────────┐
│  Phase 1: DEEP CONTEXT BUILDING (deep-context.md)               │
│  ┌─ 1A: Orientation → structural scan, establish anchors        │
│  ├─ 1B: Deep Analysis → line-by-line micro-analysis             │
│  └─ 1C: Synthesis → invariants, state map, risk clusters        │
│  ⛔ COMPLETENESS GATE — must pass before proceeding             │
├─────────────────────────────────────────────────────────────────┤
│  Phase 2: RECONNAISSANCE (reconnaissance.md)                    │
│  → Languages, frameworks, entry points, deps, secrets, auth     │
├─────────────────────────────────────────────────────────────────┤
│  Phase 3: THREAT MODEL (threat-model.md)                        │
│  → Architecture, trust boundaries, STRIDE, attacker profiles    │
├─────────────────────────────────────────────────────────────────┤
│  Phase 4: VULNERABILITY HUNTING (vulnerability-hunting.md)      │
│  → OWASP Top 10 systematic check with grep + manual review     │
├─────────────────────────────────────────────────────────────────┤
│  Phase 5: DEEP DIVE (vulnerability-hunting.md, Phase 5)         │
│  → Full exploit scenarios for HIGH/CRITICAL findings            │
├─────────────────────────────────────────────────────────────────┤
│  Phase 6: REPORT (reporting.md)                                 │
│  → Structured markdown report with all findings                 │
└─────────────────────────────────────────────────────────────────┘
```

## Workflow: Diff Mode

```
┌─────────────────────────────────────────────────────────────────┐
│  Step 1: INTAKE & CONTEXT (diff-review.md + deep-context.md)    │
│  → Extract diff, risk-score files, comprehend changed functions │
├─────────────────────────────────────────────────────────────────┤
│  Step 2: CHANGED CODE ANALYSIS (diff-review.md)                 │
│  → BEFORE/AFTER per hunk, git history, blast radius, tests      │
├─────────────────────────────────────────────────────────────────┤
│  Step 3: ADVERSARIAL ANALYSIS (diff-review.md)                  │
│  → Exploit scenarios for HIGH risk changes                      │
├─────────────────────────────────────────────────────────────────┤
│  Step 4: REPORT (reporting.md)                                  │
│  → Structured diff review report with verdict                   │
└─────────────────────────────────────────────────────────────────┘
```

## Phase Decision Tree

**Repo mode — follow in order:**

1. Start with **reconnaissance** → read [reconnaissance.md](reconnaissance.md)
   - Detect languages, frameworks, entry points, dependencies, secrets, auth
2. Build **deep context** for key functions → read [deep-context.md](deep-context.md)
   - Depth determines which functions: QUICK (entry points), STANDARD (public), DEEP (all)
3. Create **threat model** → read [threat-model.md](threat-model.md)
   - Architecture, trust boundaries, STRIDE, attacker profiles
4. Hunt **vulnerabilities** → read [vulnerability-hunting.md](vulnerability-hunting.md) (Phase 4)
   - Systematic OWASP check informed by threat model
5. **Deep dive** on HIGH/CRITICAL → read [vulnerability-hunting.md](vulnerability-hunting.md) (Phase 5)
   - Full exploit scenarios, exploitability rating
6. Write **report** → read [reporting.md](reporting.md)

**Diff mode — follow in order:**

1. **Intake & triage** → read [diff-review.md](diff-review.md) (Section 1-2)
   - Extract changes, risk-score files, build context on changed functions using [deep-context.md](deep-context.md)
2. **Analyze changes** → read [diff-review.md](diff-review.md) (Section 3-6)
   - BEFORE/AFTER, git history, test coverage, blast radius
3. **Adversarial analysis** for HIGHs → read [diff-review.md](diff-review.md) (Section 7-8)
   - Exploit scenarios, cross-cutting pattern checks
4. Write **report** → read [reporting.md](reporting.md) (Diff mode template)

## Quality Checklist

Before finalizing any review, verify:

- [ ] Every finding has a file:line reference
- [ ] Every HIGH/CRITICAL has an exploit scenario with concrete steps
- [ ] Threat model covers all trust boundaries discovered in reconnaissance
- [ ] Dependencies were checked (manifests + lock files)
- [ ] Auth/authz coverage was mapped (which endpoints lack protection)
- [ ] Confidence level stated for each finding (HIGH/MEDIUM/LOW confidence)
- [ ] Coverage stated: what was reviewed, what was skipped, why
- [ ] No severity inflation: MED findings aren't dressed up as HIGH
- [ ] No severity deflation: HIGH findings aren't minimized for convenience
- [ ] Report follows template in [reporting.md](reporting.md)
- [ ] (Diff mode) Blast radius calculated for each modified function
- [ ] (Diff mode) Git blame checked for security-relevant removed code

## Subagent Strategy

Spawn subagents to parallelize work and protect context window. Rules:

### When to Spawn

| Situation | Subagent Task | Why |
|-----------|--------------|-----|
| Phase 1B: Dense function (>50 LOC, multiple branches + external calls) | Full micro-analysis of that function | Prevents context blowup in main thread |
| Phase 1B: Long call chain (>3 hops) | Trace the chain end-to-end, return summary | Call chain context is easy to lose |
| Phase 1B: Crypto or math-heavy logic | Focused analysis of the algorithm | Requires undivided attention |
| Phase 2: Large codebase (>200 files) | Parallel recon: one for deps, one for entry points, one for secrets | Speed up enumeration |
| Phase 4: Multiple OWASP categories to check | One subagent per OWASP category (A01-A10) | Parallelizes the grep + review cycle |
| Phase 5: Multiple HIGH/CRITICAL findings | One subagent per finding for exploit scenario development | Each exploit trace needs full attention |
| Diff mode: Large PR (50+ files) | Parallel risk-scoring and context building by file group | Prevents bottleneck on sequential file review |

### Subagent Rules

1. Every subagent **must** follow the same methodology as the main agent (same micro-analysis rules, same anti-hallucination rules, same quality thresholds).
2. Subagent output **must** be structured — use the templates from the relevant phase doc.
3. Main agent **must** verify subagent output connects to the global model before integrating. Never blindly merge.
4. Subagent findings **must** include file:line references. No vague summaries.
5. If a subagent discovers something that contradicts the main agent's model, the main agent must resolve the contradiction explicitly.

## When to Use

- Full repository security audit (any language)
- PR or commit security review before merge
- Threat modeling a new or unfamiliar codebase
- Pre-engagement reconnaissance for a penetration test
- Security review before a major release
- Evaluating a third-party library or open-source dependency

## When NOT to Use

- **Documentation-only changes** — No security impact; skip the review
- **Formatting/style-only changes** — Linting, whitespace, import ordering; skip
- **You need automated SAST** — This is manual review methodology; use Semgrep/CodeQL for automated scanning, then use this skill to triage results
- **Smart contract audit** — Use Trail of Bits' dedicated Solidity/blockchain skills instead
- **Compliance checklist** — This is technical security review, not SOC2/HIPAA compliance mapping

## Example Usage

### Repo Mode
```
/security-review /path/to/my-api --depth standard
```
Reviews the full `my-api` codebase at standard depth. Produces `MY-API_SECURITY_REVIEW_2025-01-15.md`.

### Diff Mode
```
/security-review abc1234..def5678 --baseline main
```
Reviews changes between commits `abc1234` and `def5678` against `main`. Produces `PROJECT_DIFF_REVIEW_2025-01-15.md`.

### Diff Mode (PR URL)
```
/security-review https://github.com/org/repo/pull/42
```
Reviews PR #42. Auto-detects diff mode. Produces a diff review report.
