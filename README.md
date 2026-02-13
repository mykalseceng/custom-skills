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
| [codebase-security-review](plugins/codebase-security-review/) | Full-pipeline security review — deep code comprehension, reconnaissance, STRIDE threat modeling, OWASP Top 10 vulnerability hunting, and exploit scenario building — for entire codebases or code changes. Supports Python, JavaScript/TypeScript, Go, Java, Rust, Ruby, PHP, and C/C++. |
| [threat-model](plugins/threat-model/) | Intent-driven threat modeling — developer intent analysis, assumption mapping, logic flaw taxonomy, and likelihood-ranked threats. Discovers threats from broken assumptions and logic flaws rather than fixed frameworks. |

## Contributing

See [CLAUDE.md](CLAUDE.md) for skill authoring guidelines.

## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).

## Acknowledgments

Structure and conventions modeled after [Trail of Bits Skills](https://github.com/trailofbits/skills).
