# Codebase Security Review

Deep code comprehension + security analysis with exploit scenario building for any codebase or code change. Language-agnostic.

**Author**: dark_knight

## Two Modes

| Mode | Input | Use Case |
|------|-------|----------|
| **Repo** | Path to repository | Full codebase security audit |
| **Diff** | PR URL, commit SHA, or diff file | Security review of code changes before merge |

## Installation

### Add the Marketplace

```
/plugin marketplace add mykalseceng/custom-skills
```

### Browse and Install

```
/plugin menu
```

## Quick Start

### Full Codebase Review

```
/security-review /path/to/repo --depth standard
```

Runs all 6 phases: deep context → reconnaissance → threat model → vulnerability hunting → deep dive → report.

### PR / Commit Review

```
/security-review https://github.com/org/repo/pull/42
/security-review abc1234..def5678 --baseline main
```

Runs diff-specific phases: intake → context building → change analysis → blast radius → adversarial analysis → report.

## Review Depths

| Depth | When | Scope |
|-------|------|-------|
| `quick` | <20 files, time-constrained | Entry points + high-risk functions |
| `standard` | Typical review | All public/exported functions |
| `deep` | Audit engagement, high-value target | Every non-trivial function |

## Phase Overview

### Repo Mode

1. **Deep Context Building** — Line-by-line comprehension, invariant extraction, state mapping
2. **Reconnaissance** — Languages, frameworks, entry points, dependencies, secrets, auth inventory
3. **Threat Modeling** — Architecture diagram, trust boundaries, STRIDE analysis, attacker profiles
4. **Vulnerability Hunting** — Systematic OWASP Top 10 check with detection patterns + manual review
5. **Deep Dive** — Full exploit scenarios for HIGH/CRITICAL findings
6. **Report** — Structured markdown report with all findings, recommendations, and coverage statement

### Diff Mode

1. **Intake & Context** — Extract changes, risk-score files, comprehend changed functions
2. **Change Analysis** — BEFORE/AFTER per hunk, git history, blast radius, test coverage
3. **Adversarial Analysis** — Exploit scenarios for HIGH risk changes
4. **Report** — Diff review report with APPROVE/REJECT/CONDITIONAL verdict

## Documentation Structure

```
skills/codebase-security-review/
  SKILL.md                  # Main skill: modes, principles, decision tree, rationalizations
  deep-context.md           # Phase 1: line-by-line code comprehension methodology
  reconnaissance.md         # Phase 2: language/framework detection, entry points, secrets, deps
  threat-model.md           # Phase 3: architecture, trust boundaries, STRIDE, attacker profiles
  vulnerability-hunting.md  # Phase 4-5: OWASP vuln hunt + deep dive with exploit scenarios
  diff-review.md            # Diff mode: git-aware review of PRs/commits
  reporting.md              # Report templates for both modes
```

## Output

Reports are written as markdown files:
- Repo mode: `<PROJECT>_SECURITY_REVIEW_<DATE>.md`
- Diff mode: `<PROJECT>_DIFF_REVIEW_<DATE>.md`

## Related Skills

For reference, this skill was inspired by:
- [Trail of Bits differential-review](https://github.com/trailofbits/claude-code-plugins) — Security-focused diff review
- [Trail of Bits audit-context-building](https://github.com/trailofbits/claude-code-plugins) — Deep code comprehension for audits
