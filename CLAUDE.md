# Contributing Skills

## Resources

**Official Anthropic documentation (always check these first):**

- [Claude Code Plugins](https://docs.anthropic.com/en/docs/claude-code/plugins)
- [Agent Skills](https://docs.anthropic.com/en/docs/claude-code/skills)
- [Best Practices](https://docs.anthropic.com/en/docs/claude-code/skills#best-practices)

**Reference:** [Trail of Bits Skills](https://github.com/trailofbits/skills) — the structure and conventions in this repo are modeled after theirs.

## Plugin Structure

```
plugins/
  <plugin-name>/
    .claude-plugin/
      plugin.json         # Plugin metadata (name, version, description, author)
    commands/             # Optional: slash commands
    skills/               # Optional: knowledge/guidance
      <skill-name>/
        SKILL.md          # Entry point with frontmatter
        references/       # Optional: detailed docs
    README.md             # Plugin documentation
```

**Important**: Component directories (`skills/`, `commands/`) must be at the plugin root, NOT inside `.claude-plugin/`. Only `plugin.json` belongs in `.claude-plugin/`.

## Frontmatter

```yaml
---
name: skill-name              # kebab-case, max 64 chars
description: "Third-person description of what it does and when to use it"
allowed-tools:                # Optional: restrict to needed tools only
  - Read
  - Grep
---
```

## Naming Conventions

- **kebab-case**: `codebase-security-review`, not `codebaseSecurityReview`
- **Avoid vague names**: `helper`, `utils`, `tools`, `misc`

## Path Handling

- Use `{baseDir}` for paths, **never hardcode** absolute paths
- Use forward slashes (`/`) even on Windows

## Quality Standards

### Required Sections

Every SKILL.md must include:

```markdown
## When to Use
[Specific scenarios where this skill applies]

## When NOT to Use
[Scenarios where another approach is better]
```

### Security Skills

For audit/security skills, also include:

```markdown
## Rationalizations to Reject
[Common shortcuts or rationalizations that lead to missed findings]
```

### Content Organization

- Keep SKILL.md **under 500 lines** — split into `references/`, supporting `.md` files
- Use **progressive disclosure** — quick start first, details in linked files
- **One level deep** — SKILL.md links to files, files don't chain to more files

## PR Checklist

Before submitting:

**Technical:**
- [ ] Valid YAML frontmatter with `name` and `description`
- [ ] Name is kebab-case
- [ ] All referenced files exist
- [ ] No hardcoded paths (`/Users/...`, `/home/...`)

**Quality:**
- [ ] Description triggers correctly (third-person, specific)
- [ ] "When to use" and "When NOT to use" sections present
- [ ] Explains WHY, not just WHAT

**Documentation:**
- [ ] Plugin has README.md
- [ ] Added to root README.md table
- [ ] Registered in marketplace.json

**Version updates (for existing plugins):**
- [ ] Increment version in both `plugin.json` and `.claude-plugin/marketplace.json`
- [ ] Ensure version numbers match between `plugin.json` and `.claude-plugin/marketplace.json`
