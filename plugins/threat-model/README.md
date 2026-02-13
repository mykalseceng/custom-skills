# Threat Model

Intent-driven threat modeling that discovers threats organically from code analysis rather than forcing them through fixed frameworks like STRIDE. Analyzes what developers intended, what they assumed, where logic diverges from intent, and how attackers exploit the gaps.

**Author**: dark_knight

## How It Differs from `codebase-security-review`

| Aspect | `threat-model` | `codebase-security-review` |
|--------|---------------|---------------------------|
| **Focus** | Developer intent, assumptions, logic flaws | OWASP vulnerabilities, exploit scenarios |
| **Approach** | Reasoning about code logic | Pattern-matching + manual review |
| **Threat source** | Emergent from broken assumptions | STRIDE framework per trust boundary |
| **Ranking** | Likelihood-first (who exploits what?) | Severity-first (how bad is it?) |
| **Output** | Ranked threat model document | Security review with findings + PoCs |

Use `threat-model` to understand **what can go wrong and why**. Use `codebase-security-review` to find **specific exploitable vulnerabilities**.

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

```
/threat-model /path/to/repo --depth standard
```

## Phase Pipeline

```
Phase 1: RECONNAISSANCE       -> What does the system do?
Phase 2: DEVELOPER INTENT      -> What did devs intend? What did they assume?
Phase 3: LOGIC FLAW HUNTING    -> Where does logic diverge from intent?
Phase 4: THREAT ANALYSIS       -> Who exploits what, ranked by likelihood?
Phase 5: REPORT                -> Ranked threat model document
```

## Review Depths

| Depth | When | Scope |
|-------|------|-------|
| `quick` | <20 files, time-constrained | Entry points + critical workflows only |
| `standard` | Typical review | All public functions + key internal paths |
| `deep` | Audit engagement, high-value target | Every non-trivial function + all assumption chains |

## Output

Reports are written as markdown: `<PROJECT>_THREAT_MODEL_<YYYY-MM-DD>.md`

## Documentation Structure

```
skills/threat-model/
  SKILL.md                 # Hub: principles, pipeline, decision tree
  reconnaissance.md        # Phase 1: structural understanding + workflow discovery
  developer-intent.md      # Phase 2: intent reconstruction + assumption mapping
  logic-flaw-taxonomy.md   # Phase 3: 12 logic flaw categories + adversarial walkthroughs
  threat-analysis.md       # Phase 4: threat derivation + likelihood-based ranking
  reporting.md             # Phase 5: report template
```
