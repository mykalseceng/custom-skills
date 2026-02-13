---
name: threat-model
description: "Run an intent-driven threat model on a codebase"
argument-hint: "<repo-path> [--depth quick|standard|deep]"
allowed-tools:
  - Read
  - Write
  - Grep
  - Glob
  - Bash
---

# Threat Model

**Arguments:** $ARGUMENTS

Parse arguments:
1. **Target** (required): path to the repository or directory to threat-model
2. **Depth** (optional): `--depth quick|standard|deep` (defaults to `standard`)

Validate:
- Target must be a valid directory path
- If target is missing, ask the user

## Execution

Load the `threat-model` skill (SKILL.md) and execute the full 5-phase pipeline:

1. **Read SKILL.md** — understand core principles, pipeline, and anti-hallucination rules
2. **Execute Phase 1** — read `reconnaissance.md`, announce phase, perform recon, output summary
3. **Execute Phase 2** — read `developer-intent.md`, announce phase, analyze intent + assumptions, pass completeness gate
4. **Execute Phase 3** — read `logic-flaw-taxonomy.md`, announce phase, hunt logic flaws across 12 categories
5. **Execute Phase 4** — read `threat-analysis.md`, announce phase, derive and rank threats by likelihood
6. **Execute Phase 5** — read `reporting.md`, announce phase, write the final report to `<PROJECT>_THREAT_MODEL_<DATE>.md`

**Phase announcement format** — output this at the start of each phase:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Phase N/5: PHASE NAME
  → Brief description of what this phase does
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Report output** — the final deliverable is a markdown file written to the target project's root directory. After writing, inform the user of: file path, threat counts by risk level, top 3 most likely threats, and top recommendation.
