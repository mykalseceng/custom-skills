# Phase 4: Threat Analysis & Ranking

Synthesize assumptions (Phase 2) and logic flaws (Phase 3) into ranked threats. Threats are **derived** from the analysis — not imposed from a checklist. Every threat must trace back to a specific broken assumption or logic flaw.

**Prerequisite**: Phases 1-3 must be complete. This phase uses:
- Phase 1: Architecture, workflows, state machines
- Phase 2: Assumption inventory (UNVALIDATED/VIOLATED), intent-reality gaps, broken chains
- Phase 3: Logic flaw inventory, compound flaws

---

## Threat Actor Profiles

Define who might exploit the threats you've found. Link each actor to which **types of assumptions** they can violate.

| Actor | Access Level | Can Violate | Primary Target |
|-------|-------------|-------------|---------------|
| **Unauthenticated external** | Public endpoints only | INPUT assumptions (malformed data), TRUST assumptions (impersonation) | Auth bypass, data theft, abuse of public functionality |
| **Authenticated user** | Own account + user endpoints | CALLER assumptions (unauthorized actions), ORDERING assumptions (race conditions), TRUST assumptions (privilege boundaries) | Privilege escalation, access other users' data, business logic abuse |
| **Privileged user (admin)** | Admin + user endpoints | ENVIRONMENT assumptions (config manipulation), TRUST assumptions (internal data trust) | Data exfiltration, backdoor installation, audit log manipulation |
| **Compromised internal service** | Internal APIs, shared resources | CALLER assumptions (trusted-caller bypass), ENVIRONMENT assumptions (service credentials) | Lateral movement, data access across service boundaries |
| **Supply chain / dependency** | Build pipeline, runtime libraries | ENVIRONMENT assumptions (safe dependency behavior), TRUST assumptions (library correctness) | RCE via malicious code, data exfiltration through dependency |

For each actor relevant to this codebase:
- Which specific UNVALIDATED assumptions can they target?
- Which logic flaws (#1-#12) are reachable from their access level?
- What's their highest-value target in this system?

---

## Trust Boundary Synthesis

Combine Phase 1 architecture with Phase 2 assumption mapping to produce trust boundaries.

### Building Trust Boundaries

For each transition between trust levels identified in Phase 2 (TRUST assumptions):

```markdown
### Boundary: [name]

**Location**: [file:line or architectural description]
**Untrusted side**: [what's on this side]
**Trusted side**: [what's on this side]

**Assumptions at this boundary**:
- [A-NNN]: [assumption] — [VALIDATED/UNVALIDATED/VIOLATED]
- [A-NNN]: [assumption] — [status]

**Logic flaws at this boundary**:
- [LF-NNN]: [flaw] — [category] (maps to F-NNN in report)

**Data crossing**: [what data moves across this boundary]
**Validation present**: [what's checked]
**Validation absent**: [what's NOT checked]
```

---

## Threat Derivation

Threats are NOT generated from a checklist. Each threat is derived from one or more:
- **UNVALIDATED assumption** → "What if this assumption is false?"
- **VIOLATED assumption** → "This assumption IS false — what's the impact?"
- **Logic flaw** → "How does an attacker exploit this flaw?"
- **Broken assumption chain** → "What cascading failure does this chain break enable?"
- **Compound flaw** → "What does the combination of these flaws unlock?"

### Derivation Process

For each source (unvalidated assumption, violated assumption, logic flaw, broken chain, compound flaw):

1. **Identify the source**: which assumption/flaw/chain
2. **Identify the actor**: which threat actor can reach this
3. **Construct the threat**: what can the actor do by exploiting this source
4. **Assess the impact**: what's the concrete consequence
5. **Score the likelihood**: using the 5-factor model below

### Finding Template

Each finding consolidates the full chain: intent gap (G-NNN from Phase 2) + logic flaw (LF-NNN from Phase 3) + threat assessment into a single unit. Use F-NNN IDs — these carry directly into the report.

```markdown
### F-NNN: [finding title]

**Source**:
- Assumption(s): [A-NNN — description]
- Logic flaw(s): [LF-NNN — description]
- Broken chain(s): [C-NNN — description]

**Actor**: [which threat actor]
**Trust boundary**: [which boundary this crosses]

**Threat scenario**:
[2-4 sentences describing what the attacker does and what happens]

**Impact**: [concrete consequence — data loss, unauthorized access, financial loss, etc.]

**Likelihood score**: [see scoring below]
**Impact level**: [LOW / MEDIUM / HIGH / CRITICAL]
**Overall risk**: [from the ranking formula]
```

---

## Likelihood-Based Ranking

The core innovation: rank by **likelihood of exploitation**, not just impact. A trivially exploitable MEDIUM-impact threat matters more than a theoretical CRITICAL-impact threat.

### 5 Likelihood Factors

| Factor | 1 (Low) | 3 (Medium) | 5 (High) |
|--------|---------|-----------|----------|
| **Attack Surface Accessibility** | Requires internal network access or pre-existing compromise | Requires authenticated access | Reachable from public internet unauthenticated |
| **Complexity** | Requires chaining 3+ flaws, precise timing, or deep technical knowledge | Requires moderate skill or 2-flaw chain | Single request, simple manipulation, copy-paste exploit |
| **Assumption Fragility** | Assumption is partially validated with edge cases | Assumption is validated in some code paths but not all | Assumption is completely UNVALIDATED with no checks anywhere |
| **Attacker Motivation** | Low-value target, no financial/data incentive | Moderate value (user data, preferences) | High value (credentials, payment data, PII, admin access) |
| **Discoverability** | Requires source code access or deep reverse engineering | Discoverable through normal usage with attention | Obvious from API docs, error messages, or basic fuzzing |

### Scoring Formula

```
Likelihood = (Accessibility × 2) + (Complexity × 2) + (Fragility × 1.5) + (Motivation × 1) + (Discoverability × 1)
             ───────────────────────────────────────────────────────────────────────────────────────────────────────
                                                         7.5

Scale:
  1.0 - 2.0  → LOW likelihood
  2.1 - 3.5  → MEDIUM likelihood
  3.6 - 5.0  → HIGH likelihood
```

### Risk Matrix

Combine likelihood with impact:

```
              │ LOW Impact  │ MED Impact  │ HIGH Impact │ CRIT Impact │
──────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
HIGH Likely   │ MEDIUM      │ HIGH        │ CRITICAL    │ CRITICAL    │
MED Likely    │ LOW         │ MEDIUM      │ HIGH        │ CRITICAL    │
LOW Likely    │ LOW         │ LOW         │ MEDIUM      │ HIGH        │
```

### Sorting Rule

**Sort findings by likelihood FIRST, then by impact within the same likelihood tier.** This guides the analyst to focus on what's most likely to be exploited, not what's theoretically worst.

---

## Attack Path Mapping

Connect individual findings into multi-step attack chains. Attack paths show how an attacker progresses from initial access to final impact.

### Path Construction

For each HIGH-likelihood finding, ask: "What can the attacker do AFTER exploiting this?" Chain findings together.

```markdown
### Attack Path: [name]

**Actor**: [threat actor]
**Entry**: [starting point]

**Steps**:
1. **F-NNN** [finding title] → gains [access/data/capability]
   - Likelihood: [score]
2. **F-NNN** [finding title] → escalates to [next level]
   - Likelihood: [score]
3. **F-NNN** [finding title] → achieves [final impact]
   - Likelihood: [score]

**Cumulative likelihood**: [product of individual likelihoods — approximate]
**Final impact**: [concrete end-state for the attacker]
```

### Path Prioritization

Rank attack paths by:
1. Cumulative likelihood (higher = worse)
2. Final impact severity
3. Number of steps (fewer = more likely to be executed completely)

---

## Focus Areas

After scoring and ranking, identify the most important areas for remediation.

### Top Findings

List the top 10 findings by risk (likelihood × impact), with:
- Finding ID and title
- Source assumptions/flaws
- Actor and scenario (1-sentence)
- Likelihood and impact scores

### Top Attack Paths

List the top 5 attack paths, with:
- Path name
- Step count and cumulative likelihood
- Final impact

### Unresolved Assumptions

List all UNVALIDATED assumptions that haven't yet been connected to a specific finding. These are potential blind spots.

---

## Phase 4 Output Template

```markdown
## Threat Analysis Summary

### Finding Inventory
| # | Finding | Source | Actor | Likelihood | Impact | Risk |
|---|---------|--------|-------|-----------|--------|------|
| F-001 | [title] | A-001, LF-003 | Authenticated user | HIGH (4.2) | HIGH | CRITICAL |
| F-002 | [title] | A-005 | Unauthenticated | MEDIUM (3.1) | CRITICAL | CRITICAL |
| F-003 | [title] | LF-007, C-002 | Authenticated user | HIGH (3.8) | MEDIUM | HIGH |

### Likelihood Factor Breakdown (for top findings)
| Finding | Accessibility | Complexity | Fragility | Motivation | Discoverability | Score |
|---------|-------------|-----------|-----------|-----------|----------------|-------|
| F-001 | 5 | 5 | 4 | 5 | 4 | 4.2 |

### Attack Paths
| # | Path | Steps | Cumulative Likelihood | Final Impact |
|---|------|-------|--------------------|-------------|
| AP-001 | [name] | 3 | [score] | [impact] |

### Trust Boundaries
| # | Boundary | UNVALIDATED Assumptions | Findings | Risk |
|---|----------|----------------------|----------|------|
| TB-001 | [name] | [count] | F-NNN, F-NNN | [level] |

### Top 10 Findings (sorted by likelihood, then impact)
1. **F-NNN**: [title] — Likelihood [score], Impact [level]
   Source: [assumptions/flaws]. Scenario: [1 sentence]
2. ...

### Top 5 Attack Paths
1. **AP-NNN**: [name] — [steps] steps, Final impact: [description]
2. ...

### Unresolved Assumptions (not connected to findings)
| # | Assumption | Type | Status | Potential Concern |
|---|-----------|------|--------|------------------|
| A-NNN | [description] | [type] | UNVALIDATED | [why it might matter] |

### Signals for Phase 5 (Report)
- Total findings: [n] (CRITICAL: [n], HIGH: [n], MEDIUM: [n], LOW: [n])
- Total attack paths: [n]
- Coverage: [what workflows were fully analyzed, what was skipped]
```
