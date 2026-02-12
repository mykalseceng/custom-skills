# Phase 2: Reconnaissance

Discover what the codebase is, what it's built with, and where the interesting attack surface lives. This phase is systematic enumeration — not analysis.

## 1. Language & Framework Detection

Detect from manifest files first, then confirm with code patterns.

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
| `*.csproj`, `*.sln` | C# | .NET |

### Framework Grep Patterns

```bash
# Python
grep -r "from flask\|from django\|from fastapi\|from starlette\|from aiohttp\|from tornado" --include="*.py"

# JavaScript/TypeScript
grep -r "require('express')\|from 'express'\|from 'fastify'\|from 'koa'\|from 'hapi'\|from 'next'" --include="*.{js,ts,jsx,tsx}"

# Go
grep -r "\"net/http\"\|\"github.com/gin-gonic\|\"github.com/gorilla\|\"github.com/labstack/echo\|\"github.com/gofiber" --include="*.go"

# Java
grep -r "import org.springframework\|import javax.servlet\|import io.micronaut\|import io.quarkus" --include="*.java"

# Ruby
grep -r "require 'sinatra'\|require 'rails'\|class.*< ApplicationController" --include="*.rb"

# PHP
grep -r "use Illuminate\|use Symfony\|use Laravel" --include="*.php"
```

## 2. Directory Structure Mapping

Map standard directories and flag non-standard layout:

| Directory | Expected Contents | Security Relevance |
|-----------|------------------|-------------------|
| `src/`, `lib/`, `app/` | Core application code | Primary review target |
| `cmd/`, `bin/` | Entry points, CLI | Entry point enumeration |
| `api/`, `routes/`, `controllers/` | HTTP handlers | Attack surface |
| `middleware/` | Request pipeline | Auth/authz enforcement |
| `models/`, `entities/`, `schemas/` | Data models | Data validation, ORM injection |
| `config/`, `settings/` | Configuration | Secrets, feature flags |
| `deploy/`, `infra/`, `terraform/`, `k8s/` | Infrastructure | Misconfigs, exposed services |
| `test/`, `tests/`, `spec/`, `__tests__/` | Test code | Test coverage signals |
| `migrations/`, `db/` | Database schemas | Schema design, privilege |
| `public/`, `static/`, `assets/` | Served files | XSS, path traversal |
| `scripts/`, `tools/` | Utility scripts | Hardcoded creds, insecure ops |

Flag anything unexpected: binary files in source directories, large data files, compiled artifacts checked in.

## 3. Config & Secrets Discovery

### Config File Patterns

```bash
# Find all config files
find . -name "*.env*" -o -name "*.yaml" -o -name "*.yml" -o -name "*.toml" -o -name "*.ini" -o -name "*.conf" -o -name "*.cfg" -o -name "*.json" | grep -vi node_modules | grep -vi vendor
```

### Secret Detection Patterns

```bash
# API keys and tokens
grep -rn "api[_-]?key\|api[_-]?secret\|api[_-]?token" --include="*.{py,js,ts,go,java,rb,php,yaml,yml,toml,json,env}" -i

# Passwords and credentials
grep -rn "password\s*=\|passwd\s*=\|secret\s*=\|credential" --include="*.{py,js,ts,go,java,rb,php,yaml,yml,toml,json,env}" -i

# Connection strings
grep -rn "postgres://\|mysql://\|mongodb://\|redis://\|amqp://\|sqlite:" --include="*.{py,js,ts,go,java,rb,php,yaml,yml,toml,json,env}"

# AWS / cloud keys
grep -rn "AKIA[0-9A-Z]\{16\}\|aws_secret\|aws_access" -i

# Private keys
grep -rn "BEGIN.*PRIVATE KEY\|BEGIN RSA\|BEGIN EC\|BEGIN DSA"

# JWT secrets
grep -rn "jwt[_-]?secret\|signing[_-]?key\|HS256\|RS256" -i --include="*.{py,js,ts,go,java,rb,php,yaml,yml,toml,json,env}"
```

### .gitignore Coverage Check

Verify these are in `.gitignore`:
- `.env`, `.env.*`
- `*.pem`, `*.key`, `*.p12`
- Config files with secrets
- Build artifacts with embedded config

Flag any secret-containing files NOT in `.gitignore`.

## 4. Dependency Inventory

### Manifest + Lock File Map

| Language | Manifest | Lock File | Check |
|----------|----------|-----------|-------|
| JavaScript | `package.json` | `package-lock.json` / `yarn.lock` / `pnpm-lock.yaml` | Lock file exists and is committed |
| Python | `pyproject.toml` / `requirements.txt` | `uv.lock` / `poetry.lock` / `requirements.txt` (pinned) | Deps are pinned to exact versions |
| Go | `go.mod` | `go.sum` | `go.sum` committed |
| Rust | `Cargo.toml` | `Cargo.lock` | Lock committed for binaries |
| Java | `pom.xml` / `build.gradle` | — | Version ranges checked |
| Ruby | `Gemfile` | `Gemfile.lock` | Lock committed |
| PHP | `composer.json` | `composer.lock` | Lock committed |

### Dependency Red Flags

- Very old lock file (check git blame for last update)
- Hundreds of dependencies for a simple application
- Dependencies pulled from non-standard registries
- Git dependencies pinned to branch instead of SHA
- No lock file at all (non-reproducible builds)

## 5. Entry Point Identification

### HTTP/API Entry Points

```bash
# Express.js
grep -rn "app\.\(get\|post\|put\|delete\|patch\|all\|use\)\s*(" --include="*.{js,ts}"
grep -rn "router\.\(get\|post\|put\|delete\|patch\|all\|use\)\s*(" --include="*.{js,ts}"

# FastAPI / Flask / Django
grep -rn "@app\.\(route\|get\|post\|put\|delete\|patch\)\|@router\.\|path(\|re_path(" --include="*.py"
grep -rn "urlpatterns" --include="*.py"

# Go net/http and frameworks
grep -rn "HandleFunc\|Handle(\|r\.GET\|r\.POST\|e\.GET\|e\.POST\|app\.Get\|app\.Post" --include="*.go"

# Java Spring
grep -rn "@RequestMapping\|@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping" --include="*.java"

# Ruby Rails
grep -rn "get '\|post '\|put '\|patch '\|delete '\|resources \|resource " --include="*.rb" config/routes.rb
```

### CLI Entry Points

```bash
# Python
grep -rn "if __name__.*__main__\|@click\.\|def main\|argparse\.\|typer\." --include="*.py"

# Go
grep -rn "func main()" --include="*.go"

# Node
grep -rn "\"main\":\|\"bin\":" package.json
```

### Message Handlers / Background Jobs

```bash
# Message queues
grep -rn "consume\|subscribe\|on_message\|handle_message\|@celery\|@task\|Bull\|BullMQ\|SQS\|pubsub" -i --include="*.{py,js,ts,go,java,rb}"

# gRPC
grep -rn "\.proto$\|grpc\.\|RegisterService\|add_.*Servicer" --include="*.{py,js,ts,go,java}"

# WebSocket
grep -rn "websocket\|ws\.\|socket\.on\|@OnMessage" -i --include="*.{py,js,ts,go,java,rb}"
```

## 6. Auth/Authz Inventory

### Auth Middleware Detection

```bash
# Middleware patterns
grep -rn "authenticate\|authorize\|isAuthenticated\|requireAuth\|protect\|guard\|@login_required\|@permission_required\|@requires_auth\|@jwt_required\|Authenticated\|AuthGuard" -i --include="*.{py,js,ts,go,java,rb,php}"

# JWT patterns
grep -rn "jwt\.\|jsonwebtoken\|jose\.\|pyjwt\|golang-jwt\|auth0" -i --include="*.{py,js,ts,go,java,rb,php}"

# Session patterns
grep -rn "session\.\(get\|set\|destroy\|regenerate\)\|express-session\|cookie-session\|flask\.session" --include="*.{py,js,ts,go,java,rb,php}"

# OAuth
grep -rn "oauth\|passport\.\|grant_type\|authorization_code\|client_credentials" -i --include="*.{py,js,ts,go,java,rb,php}"
```

### Auth Coverage Map

For each entry point found in Step 5, determine:
- Is it protected by auth middleware? (YES / NO / UNCLEAR)
- What auth mechanism? (JWT / session / API key / none)
- What authorization level? (public / authenticated / admin / specific role)

Flag endpoints that should be protected but aren't.

## Phase 2 Output Template

```markdown
## Reconnaissance Summary

### Languages & Frameworks
- Primary: [language] + [framework] ([version])
- Secondary: [if polyglot]

### Codebase Size
- Files: [count by language]
- Estimated LOC: [count]
- Directories: [key dirs and their purpose]

### Entry Points
| Route/Handler | Method | File:Line | Auth Required | Auth Mechanism |
|--------------|--------|-----------|---------------|----------------|
| `/api/users` | GET | routes/users.ts:15 | Yes | JWT |
| `/api/login` | POST | routes/auth.ts:8 | No | — |
| ... | | | | |

### Dependencies
- Total: [count]
- Lock file: [present/absent]
- Last updated: [date from git blame]
- Red flags: [any from the checklist above]

### Secrets & Config
- Secret files found: [list]
- .gitignore coverage: [adequate/gaps found]
- Hardcoded secrets: [count, locations]

### Auth Architecture
- Mechanism: [JWT / session / API key / OAuth / mixed]
- Middleware: [file:line]
- Unprotected endpoints: [list of concerning ones]

### Risk Signals for Phase 3
1. [signal] — [why it matters]
2. ...
```
