# Phase 1: Reconnaissance

Discover what the codebase is, how it's structured, and what workflows it supports. This phase is structural understanding — lighter than vulnerability-oriented recon. Its output feeds Phase 2 (Developer Intent Analysis), which uses the workflows, state machines, and entry points discovered here to decide which functions to analyze.

## 1. Language & Framework Detection

Detect from manifest files first, confirm with code patterns.

### Manifest Detection

| File | Language | Ecosystem |
|------|----------|-----------|
| `package.json` | JavaScript/TypeScript | Node.js/npm |
| `tsconfig.json` | TypeScript | Node.js |
| `pyproject.toml`, `setup.py`, `requirements.txt` | Python | pip/uv/poetry |
| `go.mod` | Go | Go modules |
| `Cargo.toml` | Rust | Cargo |
| `pom.xml`, `build.gradle` | Java/Kotlin | Maven/Gradle |
| `Gemfile` | Ruby | Bundler |
| `composer.json` | PHP | Composer |
| `CMakeLists.txt`, `Makefile` | C/C++ | CMake/Make |

### Framework Grep Patterns

```bash
# Python
grep -r "from flask\|from django\|from fastapi\|from starlette" --include="*.py"

# JavaScript/TypeScript
grep -r "require('express')\|from 'express'\|from 'fastify'\|from 'next'" --include="*.{js,ts,jsx,tsx}"

# Go
grep -r "\"net/http\"\|\"github.com/gin-gonic\|\"github.com/gorilla\|\"github.com/labstack/echo" --include="*.go"

# Java
grep -r "import org.springframework\|import javax.servlet\|import io.micronaut" --include="*.java"

# Rust
grep -r "use actix_web\|use axum\|use rocket\|use warp\|use hyper" --include="*.rs"

# Ruby
grep -r "require 'sinatra'\|require 'rails'" --include="*.rb"

# PHP
grep -r "use Illuminate\|use Symfony\|use Laravel" --include="*.php"

# C/C++
grep -rn "include.*<microhttpd\|include.*<crow\|include.*<boost/beast\|include.*<boost/asio" --include="*.{c,cpp,cc,h,hpp}"
```

## 2. Entry Point Identification

### HTTP/API Entry Points

```bash
# Express.js
grep -rn "app\.\(get\|post\|put\|delete\|patch\|all\|use\)\s*(" --include="*.{js,ts}"
grep -rn "router\.\(get\|post\|put\|delete\|patch\)\s*(" --include="*.{js,ts}"

# FastAPI / Flask / Django
grep -rn "@app\.\(route\|get\|post\|put\|delete\)\|@router\.\|path(\|re_path(" --include="*.py"

# Go
grep -rn "HandleFunc\|Handle(\|r\.GET\|r\.POST\|e\.GET\|e\.POST" --include="*.go"

# Java Spring
grep -rn "@RequestMapping\|@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping" --include="*.java"

# Rust
grep -rn "\.route\|\.resource\|#\[get\|#\[post\|Router::new" --include="*.rs"
```

### CLI & Background Entry Points

```bash
# CLI
grep -rn "if __name__.*__main__\|@click\.\|argparse\.\|typer\." --include="*.py"
grep -rn "func main()" --include="*.go"
grep -rn "fn main()\|#\[tokio::main\]" --include="*.rs"

# Message handlers / background jobs
grep -rn "consume\|subscribe\|on_message\|@celery\|@task\|Bull\|SQS\|pubsub" -i --include="*.{py,js,ts,go,java,rb,rs}"

# WebSocket
grep -rn "websocket\|ws\.\|socket\.on\|@OnMessage" -i --include="*.{py,js,ts,go,java,rb,rs}"
```

## 3. Auth Inventory

Quick enumeration of auth mechanisms — not a deep audit, just enough to understand trust structure.

```bash
# Auth middleware
grep -rn "authenticate\|authorize\|isAuthenticated\|requireAuth\|@login_required\|@jwt_required\|AuthGuard" -i --include="*.{py,js,ts,go,java,rb,php,rs}"

# JWT
grep -rn "jwt\.\|jsonwebtoken\|jose\.\|pyjwt\|golang-jwt" -i --include="*.{py,js,ts,go,java,rb,php,rs}"

# Session
grep -rn "session\.\(get\|set\|destroy\)\|express-session\|flask\.session" --include="*.{py,js,ts,go,java,rb,php,rs}"

# OAuth
grep -rn "oauth\|passport\.\|grant_type\|authorization_code" -i --include="*.{py,js,ts,go,java,rb,php,rs}"
```

For each entry point, note: protected (YES/NO/UNCLEAR), mechanism (JWT/session/API key/none), role required (public/authenticated/admin).

## 4. Workflow Discovery

Enumerate what users can **DO** with this system. Workflows are end-to-end actions, not individual endpoints.

### Identification Approach

1. **Read route files / controllers** — Group related endpoints into workflows (e.g., `/auth/login`, `/auth/register`, `/auth/forgot-password` → "Authentication workflow")
2. **Scan for domain actions** — Look for verb-heavy function names:

```bash
# Action verbs indicating workflows
grep -rn "def create_\|def update_\|def delete_\|def invite_\|def transfer_\|def approve_\|def reject_\|def cancel_\|def export_\|def import_\|def checkout\|def pay\|def refund\|def subscribe\|def unsubscribe\|def verify\|def reset_\|def promote\|def revoke" --include="*.{py,js,ts,go,java,rb,rs}"
```

3. **Check for multi-step flows** — Look for wizard/step patterns, status transitions, staged operations:

```bash
grep -rn "step\|stage\|phase\|wizard\|onboard\|setup_\|flow\|pipeline" -i --include="*.{py,js,ts,go,java,rb,rs}"
```

### Workflow Inventory Template

For each workflow discovered:

```
Workflow: [name, e.g., "User Registration"]
  Steps: [ordered list of functions/endpoints involved]
  Actors: [who initiates it — user, admin, system, cron]
  State changes: [what data is created/modified]
  Auth required: [at which steps]
  Sensitive data: [what PII/secrets/financial data flows through]
  Cross-module: [does it span multiple modules?]
```

## 5. State Machine Detection

Find enum/status/state fields and map their transitions. State machines are a rich source of logic flaws (invalid transitions, missing terminal states, race conditions on state changes).

### Detection Patterns

```bash
# Enum-like status fields
grep -rn "status\s*=\|state\s*=\|phase\s*=\|stage\s*=" --include="*.{py,js,ts,go,java,rb,rs}"
grep -rn "enum.*Status\|enum.*State\|STATUS_\|STATE_" --include="*.{py,js,ts,go,java,rb,rs}"

# Transition functions
grep -rn "transition\|set_status\|update_status\|change_state\|move_to\|advance\|progress" -i --include="*.{py,js,ts,go,java,rb,rs}"

# Finite state patterns
grep -rn "pending\|active\|inactive\|suspended\|cancelled\|completed\|expired\|draft\|published\|archived" -i --include="*.{py,js,ts,go,java,rb,rs}"
```

### State Machine Inventory Template

For each state machine found:

```
State Machine: [name, e.g., "Order Status"]
  Location: [file:line]
  States: [list all possible states]
  Transitions:
    [state_A] → [state_B]: triggered by [action], guarded by [condition]
    [state_B] → [state_C]: triggered by [action], guarded by [condition]
  Terminal states: [states with no outgoing transitions]
  Missing transitions: [states that seem to lack expected transitions]
  Guard gaps: [transitions without adequate guards]
```

## 6. Data Sensitivity Classification

Lightweight classification to guide where to focus assumption analysis in Phase 2.

| Classification | Examples | Phase 2 Priority |
|---------------|---------|-----------------|
| **CRITICAL** | Credentials, encryption keys, payment data, health records | Must analyze all functions touching this data |
| **HIGH** | PII (name, email, phone, address), session tokens, API keys | Analyze at STANDARD+ depth |
| **MEDIUM** | User preferences, non-sensitive business data, logs | Analyze at DEEP depth only |
| **LOW** | Public data, static content, documentation | Skip unless part of a sensitive workflow |

```bash
# Critical data patterns
grep -rn "password\|passwd\|secret_key\|private_key\|credit_card\|card_number\|ssn\|social_security" -i --include="*.{py,js,ts,go,java,rb,rs}"

# High data patterns
grep -rn "email\|phone\|address\|date_of_birth\|session_token\|api_key\|bearer" -i --include="*.{py,js,ts,go,java,rb,rs}" | grep -vi "test\|spec\|mock\|fixture"
```

## Phase 1 Output Template

```markdown
## Reconnaissance Summary

### Languages & Frameworks
- Primary: [language] + [framework] ([version])
- Secondary: [if polyglot]

### Codebase Size
- Files: [count by language]
- Estimated LOC: [count]

### Entry Points
| Route/Handler | Method | File:Line | Auth | Role |
|--------------|--------|-----------|------|------|
| `/api/users` | GET | routes/users.ts:15 | JWT | authenticated |
| `/api/login` | POST | routes/auth.ts:8 | none | public |

### Auth Architecture
- Mechanism: [JWT / session / API key / mixed]
- Middleware: [file:line]
- Unprotected entry points: [list]

### Workflow Inventory
| # | Workflow | Steps | Actors | Sensitive Data | Cross-Module |
|---|---------|-------|--------|---------------|-------------|
| 1 | User Registration | 3 | user | email, password | yes |
| 2 | Payment | 5 | user | card data, amount | yes |

### State Machine Inventory
| # | State Machine | Location | States | Transitions | Guard Gaps |
|---|-------------|----------|--------|-------------|-----------|
| 1 | Order Status | models/order.py:12 | 5 | 7 | 2 |

### Data Sensitivity Map
| Classification | Data Types Found | Locations |
|---------------|-----------------|-----------|
| CRITICAL | passwords, payment data | [files] |
| HIGH | PII, API keys | [files] |

### Signals for Phase 2 (Developer Intent)
1. [signal] — [which functions/workflows to prioritize]
2. ...
```
