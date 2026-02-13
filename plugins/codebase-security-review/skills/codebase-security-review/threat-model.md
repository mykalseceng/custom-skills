# Phase 3: Threat Modeling

Build an architecture-level understanding of what the system does, where the trust boundaries are, who the attackers are, and what can go wrong. This phase uses Phase 1 (Reconnaissance) data and Phase 2 (Deep Context) understanding to bridge enumeration and vulnerability hunting.

## 1. System Architecture Mapping

### Component Identification

Map every distinct component from the reconnaissance output:

```
Component: [name]
  Type:     [web server / API / database / cache / queue / external service / CDN / ...]
  Language: [from recon]
  Owner:    [which team/module owns it]
  Data:     [what data it stores/processes]
  Exposed:  [externally / internally / both]
```

### Communication Patterns

For each pair of communicating components:

```
Connection: [Component A] → [Component B]
  Protocol:  [HTTP/HTTPS / gRPC / TCP / WebSocket / message queue / shared DB / ...]
  Auth:      [mTLS / JWT / API key / session / none]
  Encryption: [TLS / none / custom]
  Data flow: [What data crosses this connection]
```

### Architecture Diagram Template

Produce an ASCII diagram showing components and their connections:

```
┌──────────┐     HTTPS/JWT     ┌──────────┐    TCP/password    ┌──────────┐
│  Client  │ ───────────────── │ API Server│ ─────────────────  │ Database │
│ (browser)│                   │ (Node.js) │                    │(Postgres)│
└──────────┘                   └──────────┘                     └──────────┘
                                    │
                               HTTPS/API key
                                    │
                               ┌──────────┐
                               │ External │
                               │  Payment │
                               │   API    │
                               └──────────┘
```

Annotate each connection with protocol, auth mechanism, and encryption status.

## 2. Trust Boundary Identification

A trust boundary exists wherever the trust level changes. Common boundaries:

| Boundary | Untrusted Side | Trusted Side | Common Vulnerabilities |
|----------|---------------|-------------|----------------------|
| Internet ↔ Load Balancer | External traffic | Internal network | DDoS, protocol attacks |
| Public ↔ Authenticated | Anonymous users | Verified users | Auth bypass, session hijacking |
| User ↔ Admin | Regular users | Admin functions | Privilege escalation, IDOR |
| Application ↔ Database | App queries | Stored data | SQL injection, ORM bypass |
| Application ↔ External API | Your code | Third-party service | SSRF, data leakage, trust assumptions |
| Frontend ↔ Backend | Client-side code | Server-side code | Input validation bypass, XSS |
| Service ↔ Service | One microservice | Another microservice | Lateral movement, auth gaps |
| User Input ↔ File System | User-supplied paths | Server filesystem | Path traversal, symlink attacks |

### Per-Boundary Template

For each boundary identified:

```markdown
### Boundary: [name]

**Location**: [file:line or architectural description]
**Untrusted side**: [what's on this side]
**Trusted side**: [what's on this side]

**Crossing mechanisms**: [How does data cross?]
  - Validation present: [YES/NO — what's checked]
  - Sanitization present: [YES/NO — what's sanitized]
  - Auth required: [YES/NO — how enforced]

**Gaps identified**:
  - [gap description]

**Risk level**: [LOW / MEDIUM / HIGH / CRITICAL]
```

## 3. Data Flow Analysis

Trace sensitive data through the system. Sensitive data includes: credentials, PII, tokens, financial data, health data, and secrets.

### Data Flow Trace Template

For each sensitive data type:

```markdown
### Data: [type, e.g., "user password"]

**Entry**: [where it enters the system — file:line]
**Processing**: [what happens to it — hashing, validation, transformation]
**Storage**: [where it's stored — DB table, file, cache]
**Transit**: [how it moves between components — encrypted? auth?]
**Logging**: [is it logged anywhere? — search grep results]
**Exit**: [where it leaves the system — API response, email, export]
```

### Data Flow Grep Patterns

```bash
# Credentials in transit
grep -rn "password\|passwd\|secret\|token\|credential\|api_key" --include="*.{py,js,ts,go,java,rb,php}" | grep -vi "test\|spec\|mock\|fixture"

# PII handling
grep -rn "email\|phone\|ssn\|social_security\|date_of_birth\|address" --include="*.{py,js,ts,go,java,rb,php}" | grep -vi "test\|spec\|mock"

# Logging of sensitive data
grep -rn "log\.\|logger\.\|console\.log\|print(\|fmt\.Print\|System\.out" --include="*.{py,js,ts,go,java,rb,php}" | grep -i "password\|secret\|token\|key\|credential"

# Encryption usage
grep -rn "encrypt\|decrypt\|cipher\|AES\|RSA\|sha256\|sha512\|bcrypt\|argon2\|scrypt\|pbkdf2" -i --include="*.{py,js,ts,go,java,rb,php}"
```

### Checks

- [ ] Passwords hashed with strong algorithm (bcrypt/argon2/scrypt) before storage
- [ ] Tokens encrypted or hashed at rest
- [ ] PII encrypted in transit (TLS)
- [ ] Sensitive data NOT in logs
- [ ] Sensitive data NOT in error messages returned to clients
- [ ] Sensitive data NOT in URLs or query parameters

## 4. Attacker Profiling

Define who might attack this system and what they can do.

| Attacker | Access Level | Interfaces | Goals | Sophistication |
|----------|-------------|-----------|-------|---------------|
| **Unauthenticated external** | None | Public endpoints, login page | Account takeover, data theft, RCE | Low-High |
| **Authenticated user** | Own account | All user-facing endpoints | Privilege escalation, access other users' data, abuse functionality | Low-Medium |
| **Admin/privileged user** | Admin panel | Admin endpoints + user endpoints | Data exfiltration, backdoor installation | Medium-High |
| **Compromised service** | Service credentials | Internal APIs, shared resources | Lateral movement, data access, pivot to other services | High |
| **Insider (developer)** | Source code, deploy pipeline | All | Backdoor, data theft, sabotage | High |
| **Supply chain** | Dependency code | Build pipeline, runtime | RCE via malicious package, data exfiltration | High |

For each attacker relevant to this codebase, detail:
- What can they access WITHOUT exploiting a vulnerability?
- What's the highest-value target they'd aim for?
- What's the most likely attack path?

## 5. STRIDE Threat Enumeration

For each trust boundary identified in Section 2, enumerate threats using STRIDE:

| Threat | Question | Example |
|--------|----------|---------|
| **S**poofing | Can an attacker pretend to be someone/something else? | Forge JWT, spoof IP, impersonate service |
| **T**ampering | Can an attacker modify data they shouldn't? | Modify request params, tamper with stored data, MITM |
| **R**epudiation | Can an attacker deny their actions? | Missing audit logs, unsigned transactions |
| **I**nformation Disclosure | Can an attacker access data they shouldn't? | Error messages, directory listing, IDOR, log leakage |
| **D**enial of Service | Can an attacker disrupt availability? | Resource exhaustion, regex DoS, unbounded queries |
| **E**levation of Privilege | Can an attacker gain higher access? | Privilege escalation, role manipulation, admin bypass |

### STRIDE Template Per Boundary

```markdown
### Boundary: [name]

| Threat | Applicable? | Scenario | Likelihood | Impact | Risk |
|--------|------------|----------|-----------|--------|------|
| Spoofing | YES/NO | [specific scenario] | L/M/H | L/M/H | [LxI] |
| Tampering | YES/NO | [specific scenario] | L/M/H | L/M/H | [LxI] |
| Repudiation | YES/NO | [specific scenario] | L/M/H | L/M/H | [LxI] |
| Info Disclosure | YES/NO | [specific scenario] | L/M/H | L/M/H | [LxI] |
| DoS | YES/NO | [specific scenario] | L/M/H | L/M/H | [LxI] |
| Elevation | YES/NO | [specific scenario] | L/M/H | L/M/H | [LxI] |
```

## 6. Risk Prioritization

### Likelihood × Impact Matrix

```
              │ LOW Impact  │ MED Impact  │ HIGH Impact │ CRIT Impact │
──────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
HIGH Likely   │ MEDIUM      │ HIGH        │ CRITICAL    │ CRITICAL    │
MED Likely    │ LOW         │ MEDIUM      │ HIGH        │ CRITICAL    │
LOW Likely    │ LOW         │ LOW         │ MEDIUM      │ HIGH        │
```

Use this matrix to prioritize which threats from the STRIDE analysis to focus on in Phase 4.

### Prioritization Output

Rank all identified threats by risk level. Phase 4 (Vulnerability Hunting) should:
- Spend most time on CRITICAL and HIGH risks
- Spot-check MEDIUM risks
- Note LOW risks but don't deep-dive unless time permits

## Phase 3 Output Template

```markdown
## Threat Model Summary

### Architecture
[ASCII diagram from Section 1]

### Trust Boundaries
| # | Boundary | Risk Level | Key Gaps |
|---|----------|-----------|----------|
| 1 | [name] | HIGH | [summary] |
| 2 | [name] | MEDIUM | [summary] |
| ... | | | |

### Sensitive Data Flows
| Data Type | Entry → Storage → Exit | Encrypted? | Logged? | Risk |
|-----------|----------------------|-----------|---------|------|
| Passwords | Login form → bcrypt → DB | Yes | No | LOW |
| API tokens | Header → memory → — | Transit only | YES(!!) | HIGH |
| ... | | | | |

### Top Threats (by risk)
| # | Threat | Boundary | STRIDE | Likelihood | Impact | Risk |
|---|--------|----------|--------|-----------|--------|------|
| 1 | [description] | [boundary] | [S/T/R/I/D/E] | HIGH | CRITICAL | CRITICAL |
| 2 | ... | | | | | |

### Attack Paths to Investigate
1. [Attacker] → [entry point] → [vulnerability type] → [impact]
2. ...

### Focus Areas for Phase 4
1. [area] — why: [connects to which threats]
2. ...
```
