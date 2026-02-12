---
name: security-review
description: "Run a security review on a codebase or code changes"
argument-hint: "<repo-path|pr-url|commit-sha|diff-path> [--mode repo|diff] [--depth quick|standard|deep] [--baseline <ref>]"
allowed-tools:
  - Read
  - Write
  - Grep
  - Glob
  - Bash
---

# Security Review

**Arguments:** $ARGUMENTS

Parse arguments:
1. **Target** (required): repo path, PR URL, commit SHA, or diff file path
2. **Mode** (optional): `--mode repo` or `--mode diff` (auto-detected if omitted)
3. **Depth** (optional): `--depth quick|standard|deep` (defaults to standard)
4. **Baseline** (optional): `--baseline <ref>` for diff mode comparison reference

Auto-detect mode if not specified:
- URL or commit SHA or `.patch`/`.diff` file → **diff mode**
- Directory path → **repo mode**
- Otherwise → ask the user

Invoke the `codebase-security-review` skill with these arguments for the full workflow.
