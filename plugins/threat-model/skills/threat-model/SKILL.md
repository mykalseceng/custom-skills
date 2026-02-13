---
name: threat-model
description: >
  Intent-driven threat modeling — developer intent analysis, assumption mapping,
  logic flaw taxonomy, and likelihood-ranked threats. Analyzes what developers
  intended, what they assumed, where logic diverges from intent, and how attackers
  exploit the gaps. Use for threat modeling any codebase where you need to understand
  what can go wrong and why, rather than hunting for specific vulnerability classes.
allowed-tools:
  - Read
  - Write
  - Grep
  - Glob
  - Bash
---

# Threat Model

Intent-driven threat modeling. Threats emerge organically from code analysis — not from fixed frameworks.

## Core Question

> What did the developer intend? Does the code achieve it? What assumptions hold it together? How can an attacker break those assumptions?

## Companion Files

All methodology docs are **sibling files in the same directory as this SKILL.md**:

- **[reconnaissance.md](reconnaissance.md)** — Phase 1: Structural understanding, workflow discovery, state machine detection
- **[developer-intent.md](developer-intent.md)** — Phase 2: Intent reconstruction, assumption mapping, gap analysis
- **[logic-flaw-taxonomy.md](logic-flaw-taxonomy.md)** — Phase 3: 12 logic flaw categories, adversarial walkthroughs
- **[threat-analysis.md](threat-analysis.md)** — Phase 4: Threat derivation, likelihood-based ranking, attack paths
- **[reporting.md](reporting.md)** — Phase 5: Report template and writing rules

## Core Principles

1. **Intent Before Judgment** — Reconstruct what the developer intended before declaring anything broken. A "bug" without understanding intent is a guess.
2. **Assumption Surfacing** — Every function rests on assumptions (about input, callers, environment, ordering, trust). Find them. Classify them. The unvalidated ones are where threats live.
3. **Logic Over Injection** — This skill focuses on reasoning flaws (broken state machines, missing edge cases, trust confusion), not injection patterns. Use `codebase-security-review` for OWASP hunting.
4. **Likelihood Over Severity** — Rank threats by how likely they are to be exploited, not how bad they'd be in theory. A MEDIUM-impact threat that's trivially exploitable matters more than a CRITICAL-impact threat requiring nation-state resources.
5. **Workflow-First** — Analyze code through the lens of user workflows (login, pay, invite, export). Isolated function analysis misses cross-function assumption chains.
6. **Evidence-Based** — Every claim needs a file:line reference or an explicit "inferred from [evidence]" marker. No hand-waving.

## Rationalizations to Reject

| Rationalization | Why It's Wrong | What to Do Instead |
|----------------|---------------|-------------------|
| "I can see what the code does, skip intent analysis" | What code does ≠ what it was meant to do. The gap is where threats live. | Reconstruct intent from docs, naming, structure, and behavior per [developer-intent.md](developer-intent.md) |
| "No obvious bugs, so it's secure" | Logic flaws are invisible to surface reading. Authorization gaps, state machine violations, and race conditions don't look like bugs. | Walk every workflow as an adversary per [logic-flaw-taxonomy.md](logic-flaw-taxonomy.md) |
| "This assumption is obviously true" | "Obviously true" assumptions are the most dangerous — nobody validates them, and attackers target them first. | Classify every assumption as VALIDATED/UNVALIDATED/VIOLATED per [developer-intent.md](developer-intent.md) |
| "Low impact, not worth modeling" | Low-impact flaws combine into high-impact attack chains. A logic flaw enabling horizontal access + a state bug enabling persistence = account takeover. | Model compound threats per [threat-analysis.md](threat-analysis.md) |
| "Just use STRIDE on each boundary" | STRIDE is a checklist, not a thinking tool. It catches spoofing/tampering but misses logic flaws that don't fit neatly into S/T/R/I/D/E. | Derive threats from broken assumptions + logic flaws, not from a fixed taxonomy |
| "The framework handles that" | Frameworks have defaults, not guarantees. Misconfigured middleware, disabled CSRF protection, overridden validators — all common. | Verify framework assumptions are actually enforced in this codebase |
| "Tests pass, so the logic is correct" | Tests verify intended behavior. Threats live in unintended behavior — the cases nobody wrote tests for. | Identify untested assumptions per [developer-intent.md](developer-intent.md) |
| "I'll just list the threats I know" | Your knowledge is a subset. Emergent threats come from the interaction of this codebase's specific assumptions, not from generic threat lists. | Let threats emerge from assumption analysis, don't force them from memory |

## Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│  Phase 1: RECONNAISSANCE (reconnaissance.md)                    │
│  → Structure, workflows, state machines, data sensitivity       │
├─────────────────────────────────────────────────────────────────┤
│  Phase 2: DEVELOPER INTENT ANALYSIS (developer-intent.md)       │
│  → Intent reconstruction, assumption mapping, gap analysis      │
│  ⛔ COMPLETENESS GATE — must pass before proceeding             │
├─────────────────────────────────────────────────────────────────┤
│  Phase 3: LOGIC FLAW HUNTING (logic-flaw-taxonomy.md)           │
│  → 12 flaw categories, adversarial workflow walkthroughs        │
├─────────────────────────────────────────────────────────────────┤
│  Phase 4: THREAT ANALYSIS & RANKING (threat-analysis.md)        │
│  → Threat derivation, likelihood ranking, attack paths          │
├─────────────────────────────────────────────────────────────────┤
│  Phase 5: REPORT (reporting.md)                                 │
│  → Ranked threat model document                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Phase Execution Protocol

**Follow in order. Do not skip phases. Read the companion file for each phase before executing it.**

At the start of each phase, output a visual phase header to the user:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Phase N/5: PHASE NAME
  → Brief description of what this phase does
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

At the end of each phase, output a brief completion summary before proceeding.

### Phase Decision Tree

1. **Phase 1: Reconnaissance** → read [reconnaissance.md](reconnaissance.md)
   - Understand what the system does: languages, entry points, workflows, state machines
   - Output: structural map + workflow inventory + state machine inventory
2. **Phase 2: Developer Intent Analysis** → read [developer-intent.md](developer-intent.md)
   - Per-function: reconstruct intent, map assumptions, find intent-reality gaps
   - Output: assumption inventory (VALIDATED/UNVALIDATED/VIOLATED) + gap analysis
   - **Must pass completeness gate before proceeding**
3. **Phase 3: Logic Flaw Hunting** → read [logic-flaw-taxonomy.md](logic-flaw-taxonomy.md)
   - Walk 12 flaw categories against the codebase; walk workflows as adversarial personas
   - Output: logic flaw inventory with file:line references
4. **Phase 4: Threat Analysis & Ranking** → read [threat-analysis.md](threat-analysis.md)
   - Derive threats from broken assumptions + logic flaws; rank by likelihood
   - Output: ranked threat list + attack paths + focus areas
5. **Phase 5: Report** → read [reporting.md](reporting.md)
   - Write the threat model document to a markdown file
   - Output: `<PROJECT>_THREAT_MODEL_<YYYY-MM-DD>.md` written to the project's root directory
   - After writing, inform the user: file path, threat counts by risk level, top 3 threats, top recommendation

## Review Depth

| Depth | When | Functions Analyzed | Assumption Depth | Threat Scope |
|-------|------|-------------------|-----------------|-------------|
| **QUICK** | <20 files, time-constrained | Entry points + critical workflow functions | Top-level assumptions only | Workflow-level threats |
| **STANDARD** | 20-200 files, typical review | All public/exported functions + key internal paths | Full assumption mapping per function | Function + workflow threats |
| **DEEP** | >200 files, audit engagement | Every non-trivial function incl. helpers | Full mapping + cross-function assumption chains | Function + workflow + compound threats |

## Quality Checklist

Before finalizing any threat model, verify:

- [ ] Every identified threat traces back to a specific broken assumption or logic flaw
- [ ] Every assumption is classified as VALIDATED / UNVALIDATED / VIOLATED with evidence
- [ ] Threats are ranked by likelihood, not just impact
- [ ] Attack paths show concrete multi-step chains, not vague "could lead to" statements
- [ ] No threat is based on a generic checklist item without codebase-specific evidence
- [ ] Coverage stated: what workflows were analyzed, what was skipped, why
- [ ] Confidence level stated for each threat (HIGH/MEDIUM/LOW confidence)
- [ ] Report follows template in [reporting.md](reporting.md)

## Anti-Hallucination Rules

These rules are mandatory throughout all phases:

1. **Never claim a threat without tracing it to a specific assumption or logic flaw.** "This could be exploited" without evidence is hallucination.
2. **Never state an assumption is VIOLATED without file:line evidence.** If you can't find evidence, classify as UNVALIDATED, not VIOLATED.
3. **Update explicitly when contradicted.** Say: "Earlier I assumed X. File:line shows X is false. Updated model: ..."
4. **"Unclear; need to inspect X" is always valid.** Never guess at intent or behavior.
5. **Distinguish verified from inferred.** "Code at file:line confirms X" vs "I infer X based on naming convention."
6. **Anchor periodically.** After every 3-5 functions, re-summarize your current assumption inventory. This prevents drift.
7. **Never reshape evidence to fit earlier assumptions.** If code contradicts your model, update the model.

## Subagent Strategy

Spawn subagents to parallelize work and protect context window:

| Situation | Subagent Task | Why |
|-----------|--------------|-----|
| Phase 1: Large codebase (>200 files) | Parallel recon: structure + workflows + state machines | Speed up enumeration |
| Phase 2: Dense function (>50 LOC, complex branching) | Full intent + assumption analysis of that function | Prevents context blowup |
| Phase 2: Long assumption chain (>3 functions) | Trace the chain end-to-end, return assumption map | Chain context is easy to lose |
| Phase 3: Multiple workflows to walk adversarially | One subagent per workflow | Parallelizes adversarial analysis |
| Phase 4: Multiple threat clusters to analyze | One subagent per cluster for derivation + ranking | Each cluster needs full attention |

### Subagent Rules

1. Every subagent **must** follow the same methodology (same analysis rules, same anti-hallucination rules, same quality thresholds).
2. Subagent output **must** be structured — use the templates from the relevant phase doc.
3. Main agent **must** verify subagent output connects to the global model before integrating.
4. Subagent findings **must** include file:line references.
5. If a subagent contradicts the main agent's model, resolve the contradiction explicitly.

## When to Use

- Threat modeling a new or unfamiliar codebase
- Understanding what can go wrong and why before a security review
- Identifying logic flaws that pattern-matching tools miss
- Mapping trust assumptions across a multi-module system
- Pre-audit threat discovery to guide where to focus vulnerability hunting
- Analyzing a system after a design change to find broken assumptions

## When NOT to Use

- **You need specific vulnerability findings with PoCs** — Use `codebase-security-review` for OWASP hunting + exploit scenarios
- **You need a diff/PR review** — Use `codebase-security-review` in diff mode
- **Documentation-only or formatting-only changes** — No threat impact
- **You need automated SAST** — Use Semgrep/CodeQL for pattern-matching, then use this skill to reason about results
- **Smart contract audit** — Use dedicated Solidity/blockchain skills
- **Compliance mapping** — This is technical threat modeling, not SOC2/HIPAA compliance

## Example Usage

```
/threat-model /path/to/my-api --depth standard
```

Runs all 5 phases. Produces `MY_API_THREAT_MODEL_2025-01-15.md`.
