## coherent-fullstack-skill

> This skill prevents these issues by establishing **contracts before code**, **type constraints as AI rules**, and **automated verification**.

# Coherent Fullstack Skill

> Make AI code a coherent full-stack software — contract-first, type-driven, convention-enforced

## What is This Skill?

**Coherent Fullstack** is a development methodology and skill designed for AI-assisted full-stack development. It solves the most common failure mode in AI coding: **frontend-backend-database misalignment**.

When you ask AI to build a full-stack app, it typically generates:
- Backend with random field names and types
- Frontend that expects completely different field names
- Database schema that doesn't match either
- APIs that return 404 or 422 errors at runtime

This skill prevents these issues by establishing **contracts before code**, **type constraints as AI rules**, and **automated verification**.

## The Core Problem

### Why AI Fullstack Projects Fall Apart

```
User Request: "Build a user profile API"

AI Backend generates:
┌─────────────────────────────┐
│ POST /auth/register        │
│ Body: { username, fullName }│
│ Returns: { userId, created }│
└─────────────────────────────┘

AI Frontend expects:
┌─────────────────────────────┐
│ POST /api/register          │
│ Body: { name, email }       │
│ Returns: { id, name, xp }    │
└─────────────────────────────┘
```

**Result**: Every API call fails. 422 errors. 404 errors. Runtime type mismatches.

This isn't a model capability problem — it's an **alignment problem**. The AI has no shared source of truth, so it generates independently in each context.

### The Three Failure Modes (from Vibe Coding Research)

| Failure Mode | Description | Consequence |
|-------------|-------------|-------------|
| **Context Loss** | AI forgets earlier decisions after many turns | Drift toward divergent implementations |
| **Assumption Drift** | AI fills gaps with reasonable defaults that compound | Field names, types, paths diverge |
| **Pattern Violations** | AI uses generic best practices instead of project conventions | Inconsistent with existing codebase |

## The 60/40 Rule

The most important shift in AI development isn't "write code faster" — it's **becoming an architecture constraint setter**.

```
┌────────────────────────────────────────────────────────────┐
│                    60/40 DEVELOPMENT RULE                  │
├────────────────────────────────────────────────────────────┤
│                                                            │
│   60% ALIGNMENT           │  40% EXECUTION                 │
│   ─────────────           │  ─────────────                 │
│   • Define boundaries     │   • AI generates code          │
│   • Write contracts       │   • Run tests                  │
│   • Set验收标准           │   • Fix regressions            │
│   • Establish patterns    │                                │
│                            │                                │
│   Once alignment fails,   │   Bottleneck shifts from      │
│   you'll pay back 10x in  │   "writing code" to            │
│   cross-service patches   │   "aligning systems"           │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### The New "Source Code" Concept

In the AI era, **real source code** isn't Python/TypeScript/Rust — it's **reviewable artifacts**:

1. **Mermaid diagrams** → structural boundaries
2. **OpenAPI / JSON Schema** → interface and type contracts
3. **English Specs / Gherkin** → behavioral constraints
4. **Planning snapshots** → compress discussions into AI-executable context

## 4-Step Workflow

### Step 1: Define Contract (Before Any Code)

```yaml
# contracts/user.yaml
User:
  id: string (UUID)
  name: string (2-50 chars)
  email: string (valid email)
  avatar_url: string | null
  created_at: ISO8601 datetime

POST /api/users:
  request: { name, email, password }
  response: { user: User, token: string }
  errors: 400 (validation), 409 (email exists)

GET /api/users/{id}:
  response: { user: User }
  errors: 404 (not found)
```

### Step 2: Implement Backend

Follow the contract. Every field must match exactly:
- JSON field names
- Types
- Optional vs required
- Error codes

### Step 3: Implement Frontend

Generate TypeScript interfaces from the contract:
```typescript
// types/user.ts
interface User {
  id: string;
  name: string;
  email: string;
  avatar_url: string | null;
  created_at: string;
}
```

### Step 4: Verify Alignment

Run automated checks:
```bash
./scripts/verify-alignment.sh
```

## When to Use This Skill

### ✅ Perfect Fit
- Building new full-stack features from scratch
- Adding backend endpoints that frontend needs
- Extending existing projects with new data models
- Any AI-assisted development where backend/frontend/database must agree

### ⚠️ Less Relevant
- Backend-only or frontend-only projects
- Projects with existing, well-documented APIs
- Simple CRUD apps with no complex relationships

## Quick Start (3 Minutes)

### 0. Install

```bash
# Option 1: Clone as standalone
git clone https://github.com/YOUR_USERNAME/coherent-fullstack-skill.git

# Option 2: Add as submodule to existing project
git submodule add https://github.com/YOUR_USERNAME/coherent-fullstack-skill.git .coherent-fullstack

# Option 3: Just copy the templates you need
cp templates/CLAUDE.md ./CLAUDE.md    # For Claude Code
cp templates/AGENTS.md ./AGENTS.md    # Cross-platform
cp templates/cursorrules ./.cursorrules  # For Cursor
```

### 1. Initialize Project Structure

```bash
# For TypeScript fullstack (recommended)
./scripts/init-project.sh --stack ts-fullstack

# For Go + React
./scripts/init-project.sh --stack go-react

# For Python FastAPI + React
./scripts/init-project.sh --stack python-react
```

### 2. Define Your First Contract

Create `contracts/api.yaml` with your first endpoint:

```yaml
openapi: 3.1.0
info:
  title: My App API
  version: 1.0.0
paths:
  /users:
    post:
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
components:
  schemas:
    CreateUserRequest:
      type: object
      required: [name, email, password]
      properties:
        name: { type: string, minLength: 2, maxLength: 50 }
        email: { type: string, format: email }
        password: { type: string, minLength: 8 }
    UserResponse:
      type: object
      properties:
        id: { type: string }
        name: { type: string }
        email: { type: string }
```

### 3. Generate Code

```bash
# Generate Go models
npx @openapitools/openapi-generator-cli generate \
  -i contracts/api.yaml \
  -g go \
  -o internal/models/

# Generate TypeScript types
npx @openapitools/openapi-generator-cli generate \
  -i contracts/api.yaml \
  -g typescript-fetch \
  -o web/src/types/
```

### 4. Verify Alignment

```bash
./scripts/verify-alignment.sh
```

## Configuration for AI Tools

### Claude Code

Copy `templates/CLAUDE.md` to your project root. Claude Code automatically reads it on session start.

**Recommended**: Install [agency-agents-zh](https://github.com/jnMetaCode/agency-agents-zh) to get 193 expert agent personas for Claude Code:

```bash
git clone https://github.com/jnMetaCode/agency-agents-zh.git
cd agency-agents-zh
./scripts/install.sh --tool claude-code
# Installs to ~/.claude/agents/
```

This gives Claude Code access to specialized roles (architect, reviewer, security auditor, etc.) that enforce the conventions defined in your CLAUDE.md.

### Cursor

Copy `templates/cursorrules` to your project root (no extension, hidden file).

### Windsurf

Copy `templates/cursorrules` to `.windsurfrules` in your project root.

### GitHub Copilot

Copy relevant sections to `.github/copilot-instructions.md`.

## Key Principles

### 1. Contract-First Development

**Always define the contract before writing implementation code.**

```bash
# ❌ WRONG: Start with backend code
echo "package models" > user.go

# ✅ RIGHT: Start with contract
echo "openapi: 3.1.0" > contracts/user.yaml
```

### 2. Type Tags as AI Contract

Go struct tags are the clearest way to communicate with AI:

```go
type User struct {
    ID        string    `json:"id" db:"id" validate:"required,uuid"`
    Name      string    `json:"name" db:"name" validate:"required,min=2,max=50"`
    Email     string    `json:"email" db:"email" validate:"required,email"`
    CreatedAt time.Time `json:"created_at" db:"created_at"`
}
```

### 3. Centralized API Layer

Never scatter API calls. One file for all frontend API interactions:

```typescript
// web/src/lib/api.ts
export const api = {
  users: {
    create: (data: CreateUserRequest) => 
      fetch('/api/users', { method: 'POST', body: JSON.stringify(data) }),
    getById: (id: string) => 
      fetch(`/api/users/${id}`),
    // ... all user endpoints
  },
  // ... other resources
};
```

### 4. Unified Error Handling

Backend and frontend must agree on error format:

```typescript
// Always return this structure
interface ApiResponse<T> {
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: Record<string, string[]>;
  };
}
```

### 5. Directory as Documentation

```
internal/
├── models/      # Data structures (single source of truth)
├── api/         # HTTP handlers (read models, write responses)
├── services/    # Business logic (no HTTP, no DB)
└── migrations/  # Database schema (version controlled)
```

### 6. Migration-First Schema

Never modify production tables directly. Always create migrations:

```sql
-- migrations/000001_add_users.up.sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- migrations/000001_add_users.down.sql
DROP TABLE users;
```

### 7. Do NOT List

**Explicit prohibitions work better than suggestions.**

```markdown
## DO NOT
- Do NOT use `any` type in TypeScript
- Do NOT return different field names than what the contract specifies
- Do NOT create database tables without migrations
- Do NOT add API endpoints without updating the contract
- Do NOT use camelCase for JSON in Go structs (we use snake_case)
```

### 8. Verify, Don't Assume

Automated verification catches what manual review misses:

```bash
# Check field name alignment
./scripts/verify-alignment.sh --check-fields

# Check API route alignment
./scripts/verify-alignment.sh --check-routes

# Check migration/model alignment
./scripts/verify-alignment.sh --check-schema
```

### 9. Prefer Real Data Over Synthetic

When seeding demo data, walk the source hierarchy top-down and stop
at the first feasible row:

| # | Source | Cost | Note |
|---|--------|------|------|
| 1 | Authoritative API (OAuth) | weeks of approval | safest |
| 2 | User-uploaded CSV | $0 | compliance is the user's |
| 3 | Public-data API / scrape | $$ | mark provenance, respect ToS |
| 4 | Synthetic | $0 | only as filler, label every row |

Add a `source` enum to every entity that mixes paths and surface
it as a UI badge. Reviewers, future maintainers, and AI prompts all
benefit from being able to tell hand-curated hero rows apart from
filler. See `references/principles.md` for the full reasoning,
patterns, and dollar-cost reality check.

## Tech Stack Recommendations

Based on type contract analysis, ranked by AI friendliness:

| Score | Stack | Use Case |
|-------|-------|----------|
| ⭐⭐⭐⭐⭐ | **tRPC + Prisma + Next.js** | TypeScript fullstack, maximum type safety |
| ⭐⭐⭐⭐½ | **FastAPI + Pydantic + React** | Python backend, automatic OpenAPI |
| ⭐⭐⭐⭐ | **chi + pgx + React** | Go backend, minimal framework |
| ⭐⭐⭐ | **Rust + Tauri** | Desktop apps, performance-critical |

See `references/tech-stack-guide.md` for detailed comparison.

## Packs: commercial-ready vertical slices

The core skill above only enforces alignment. To turn an aligned project into a **shippable commercial product** you need repeatable building blocks: auth, billing, multi-tenancy, notifications, file upload, observability, admin shell.

Each block is a **pack** — a self-contained directory under `packs/` containing an OpenAPI contract, migrations, reference templates, and a `CLAUDE.md.snippet` that gets merged into the consuming project's `CLAUDE.md` so AI tools know the rules of that slice.

Install a pack:

```bash
./scripts/pack-add.sh auth          # adds contracts/, migrations/, src/auth/, web/src/auth/, CLAUDE.md block
./scripts/pack-add.sh --list        # see available packs
./scripts/pack-add.sh --remove auth # removes the CLAUDE.md block (does not delete code)
```

Re-running install is idempotent — the `CLAUDE.md` block is replaced, not duplicated.

Currently shipping (all verified via `verify-pack.sh`):

**Tier 1 — commercial must-have:**

| Pack | Depends on | Description |
|------|------------|-------------|
| `auth` | — | Registration, login, refresh, logout, password reset, email verification, JWT sessions |
| `observability` | — | Structured logs + request_id, `/healthz` `/readyz` `/metrics`, OTel hooks, Sentry-compatible reporter |
| `notifications` | — | Transactional outbox, typed template registry, retry worker, HMAC-verified delivery webhooks |
| `multitenancy` | `auth` | Organizations, memberships, RBAC, invitations, `withOrg` AsyncLocalStorage scope |
| `billing` | `auth` | Stripe checkout + customer portal + idempotent webhook receiver, plan catalog in code |

**Tier 2 — important for production:**

| Pack | Depends on | Description |
|------|------------|-------------|
| `files` | `auth` | S3/R2 presigned uploads; bytes never proxied through the app |
| `gdpr` | `auth` | Pluggable exporter/eraser registry, data export, scheduled erasure, cookie consent |
| `webhooks` | `auth` | Outbound webhooks: HMAC signing, retry policy, audit log, redelivery |
| `audit` | `auth` | Owns the `audit_log` table; any pack records via `audit.record(tx, ...)` |
| `admin` | `auth`, `audit` | Email-allowlist super-admin, user/org admin actions (records via audit pack) |
| `search` | `auth` | Postgres FTS (`tsvector` + GIN), pluggable indexer registry, ranked + paginated `/search` |

**Tier 3 — additional auth modes / rollout / cross-cutting:**

| Pack | Depends on | Description |
|------|------------|-------------|
| `auth-oauth` | `auth` | Google + GitHub OAuth with HMAC-bound state and PKCE; no auto-link |
| `auth-2fa` | `auth` | TOTP (RFC 6238), AES-encrypted secrets, recovery codes, login challenge |
| `auth-magiclink` | `auth` | Single-use 10-min email tokens for passwordless sign-in |
| `feature-flags` | `auth` | In-process flag eval with targeting rules and admin CRUD; never gate auth |
| `rate-limit` | — | Token-bucket middleware (Redis or in-memory). Convention-only — no public routes |

Together: a complete commercial-product skeleton. See [`examples/saas-mvp/README.md`](./examples/saas-mvp/README.md) for an end-to-end walkthrough.

Validate any pack — including new ones you write — with:

```bash
./scripts/verify-pack.sh <pack-name>
./scripts/verify-pack.sh --all
```

See [`PACK_SPEC.md`](./PACK_SPEC.md) for the contract every pack must satisfy.

## File Structure

```
coherent-fullstack-skill/
├── SKILL.md                    # This file
├── README.md                   # GitHub README
├── PACK_SPEC.md                # Contract every pack must satisfy
├── LICENSE                     # MIT
├── references/
│   ├── principles.md           # 8 core principles with examples
│   ├── anti-patterns.md        # Common mistakes to avoid
│   └── tech-stack-guide.md     # Stack comparison and recommendations
├── templates/
│   ├── CLAUDE.md               # Claude Code configuration
│   ├── AGENTS.md               # Cross-platform rules
│   ├── cursorrules             # Cursor rules
│   ├── project-structure/       # Directory templates by stack
│   ├── api-contract/           # Model examples by language
│   ├── migration/              # SQL migration templates
│   ├── error-handling/         # Error response examples
│   └── billing-extension/      # Stripe integration template
├── packs/                      # Commercial-ready vertical slices
│   ├── auth/                   # Registration, login, JWT, password reset, email verification
│   ├── observability/          # Logs, /metrics, /healthz, /readyz, OTel
│   ├── notifications/          # Transactional outbox + email templates
│   ├── multitenancy/           # Orgs, memberships, RBAC, invitations
│   ├── billing/                # Stripe checkout, portal, webhooks, plans
│   ├── files/                  # S3 presigned uploads
│   ├── gdpr/                   # Export, erasure, consent, exporter registry
│   ├── webhooks/               # Outbound webhooks with signing + retry
│   ├── admin/                  # Super-admin gate + audit log
│   ├── auth-oauth/             # Google + GitHub OAuth
│   ├── auth-2fa/               # TOTP 2FA
│   ├── auth-magiclink/         # Passwordless email sign-in
│   ├── feature-flags/          # In-process flag eval + targeting
│   ├── rate-limit/             # Token-bucket middleware (convention-only pack)
│   ├── audit/                  # Append-only audit_log + record() helper
│   └── search/                 # Postgres FTS dispatcher + indexer registry
├── scripts/
│   ├── init-project.sh         # Project initialization (--packs supported)
│   ├── pack-add.sh             # Install/remove a pack into the current project
│   ├── verify-pack.sh          # Validate a pack against PACK_SPEC (--deep)
│   ├── verify-alignment.sh     # Alignment verification
│   └── generate-types.sh       # Regenerate types from contracts/*.yaml across stacks
└── examples/
    ├── README.md               # Usage examples
    ├── saas-mvp/               # End-to-end SaaS bootstrap walkthrough
    └── integration-tests/      # Docker-compose runners that exercise pack contracts
        └── auth/               # 13-assertion contract conformance test for the auth pack
```

## Contributing

Contributions welcome! Please read `references/principles.md` for the philosophy behind this skill.

## License

MIT License - see `LICENSE` file.

---
> Source: [xinzhuwang-wxz/coherent-fullstack-skill](https://github.com/xinzhuwang-wxz/coherent-fullstack-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
