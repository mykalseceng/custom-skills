# Skills Marketplace

A Claude Code plugin marketplace providing skills for AI-assisted security analysis workflows.

## Installation

### Add the Marketplace

```
/plugin marketplace add <user>/custom-skills
```

### Browse and Install Plugins

```
/plugin menu
```

### Local Development

To add the marketplace locally (e.g., for testing or development), navigate to the **parent directory** of this repository:

```
cd /path/to/parent
/plugins marketplace add ./custom-skills
```

## Available Plugins

### Code Auditing

| Plugin | Description |
|--------|-------------|
| [codebase-security-review](plugins/codebase-security-review/) | Full-codebase security review with deep code comprehension, threat modeling, OWASP vulnerability hunting, and exploit scenario building. Inspired by [Trail of Bits](https://github.com/trailofbits/skills) audit-context-building and differential-review skills. |

## Contributing

See [CLAUDE.md](CLAUDE.md) for skill authoring guidelines.

## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).

## Acknowledgments

Structure and methodology inspired by [Trail of Bits Skills Marketplace](https://github.com/trailofbits/skills).
