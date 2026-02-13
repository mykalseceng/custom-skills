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

Invoke the `threat-model` skill with these arguments for the full workflow.
