## msa-claude-md

> Generate minimal (<100 line) root CLAUDE.md + per-service CLAUDE.md files for MSA projects. Auto-detects tech stacks, uses See (not @) for zero-cost context loading, domain grouping for 10+ services. Use when setting up or auditing CLAUDE.md in microservice architectures.


# MSA CLAUDE.md Generator

Generate a **minimal root CLAUDE.md** (< 100 lines) as navigation hub, plus **per-service CLAUDE.md** files. All details are doc-referenced, never inlined.

## When to Activate

- User asks to "set up CLAUDE.md" or "generate CLAUDE.md" for a project with multiple services
- User has a monorepo or MSA project and wants context-optimized Claude Code instructions
- User asks to "audit" or "fix" existing CLAUDE.md in a microservice architecture
- User mentions "context optimization" or "too many services" in relation to CLAUDE.md
- Project contains 3+ services detected by build files (pom.xml, package.json, go.mod, etc.)

## Core Philosophy

1. **Root = index, not encyclopedia** -- service registry + shared conventions only
2. **Per-service = autonomous** -- each service CLAUDE.md is self-contained
3. **Conditional loading via `See`** -- Claude reads on-demand, zero startup cost
4. **< 100 lines root** -- every line must pass: "would removing this cause Claude to make mistakes?"
5. **Context isolation** -- backend services don't need frontend guides, and vice versa

## CRITICAL: `@` vs `See` References

| Syntax | Behavior | Context Cost |
|--------|----------|-------------|
| `@docs/guide.md` | Loads into memory at startup | **Full file size** |
| `See docs/guide.md` | Claude reads on-demand when needed | **Zero until accessed** |

**ALWAYS use `See` (without `@`)** for doc references in CLAUDE.md files.

Source: [Monorepo CLAUDE.md Organization](https://dev.to/anvodev/how-i-organized-my-claudemd-in-a-monorepo-with-too-many-contexts-37k7)

## Workflow

### Phase 1: Discover MSA Structure

Scan the project to identify services and their tech stacks:

```bash
find . -maxdepth 3 \( \
  -name "pom.xml" -o -name "build.gradle*" \
  -o -name "package.json" -o -name "requirements.txt" -o -name "pyproject.toml" \
  -o -name "go.mod" -o -name "Cargo.toml" \
  -o -name "Dockerfile" -o -name "docker-compose*.yml" \
\) 2>/dev/null | head -60
```

**Stack detection rules:**

| Indicator | Backend Stack |
|-----------|--------------|
| `pom.xml` or `build.gradle` + Spring deps | Java / Spring Boot |
| `package.json` + express/nestjs/fastify | Node.js / Express or NestJS |
| `requirements.txt` or `pyproject.toml` + fastapi/django/flask | Python / FastAPI or Django |
| `go.mod` | Go |
| `Cargo.toml` | Rust |

| Indicator | Frontend Stack |
|-----------|---------------|
| `package.json` + vue | Vue.js |
| `package.json` + react or next | React / Next.js |
| `package.json` + svelte or @sveltejs | Svelte / SvelteKit |
| `package.json` + angular | Angular |
| `package.json` + nuxt | Nuxt.js |

Identify:
- Service directories and their detected tech stacks
- Shared libraries / common modules
- Infrastructure configs (docker-compose, k8s, terraform)
- API contract files (OpenAPI specs, protobuf)
- Existing documentation (docs/, README files)

### Phase 2: Generate Root CLAUDE.md (< 100 lines)

The root file is a **navigation hub only**. For 10+ services, use domain grouping.

```markdown
# <Project Name>

<One-line description of the system>

## Service Domains

### Core
| Service | Path | Stack | Port |
|---------|------|-------|------|
| `auth` | `services/auth/` | <detected stack> | <port> |
| `user` | `services/user/` | <detected stack> | <port> |

### Business
| Service | Path | Stack | Port |
|---------|------|-------|------|
| `order` | `services/order/` | <detected stack> | <port> |
| `payment` | `services/payment/` | <detected stack> | <port> |

### Frontend
| Service | Path | Stack | Port |
|---------|------|-------|------|
| `admin` | `frontends/admin/` | <detected stack> | <port> |
| `customer` | `frontends/customer/` | <detected stack> | <port> |

Full service catalog: See `docs/services.md`

## Quick Commands

| Command | Description |
|---------|-------------|
| `docker compose up -d` | Start all services |
| `<stack-specific run command>` | Start single backend service |
| `<stack-specific dev command>` | Start single frontend |

## Shared Conventions

- API specs: See `docs/api/`
- Shared libs: See `libs/common/`
- Env config: `.env.example` per service
- New service setup: See `docs/service-template.md`

## Cross-Service Patterns

- Sync: REST / gRPC -- See `docs/api-contracts.md`
- Async: Message queue -- See `docs/events.md`
- Auth: JWT / session -- See `docs/auth-flow.md`
- Observability -- See `docs/observability.md`

## Gotchas

- <critical cross-service startup order>
- <shared env vars across services>

Each service has its own `CLAUDE.md` -- auto-loaded when working in that directory.
```

**Rules:**
- **Domain grouping**: Group services by domain (Core / Business / Frontend / Infra) when > 6 services
- Service table: 4 columns only (`Service | Path | Stack | Port`)
- If > 15 services: show top 8-10, point to `docs/services.md` for full catalog
- Quick commands: max 5 rows, use detected stack commands
- All doc references use **`See` not `@`**
- **Total must be < 100 lines -- count with `wc -l`**

### Phase 2b: Generate `docs/services.md` Catalog (for 10+ services)

Full catalog with extra columns (Owner, Purpose, Deps) that don't fit in root:

```markdown
# Service Catalog

Last updated: <date>

## Domain: <Domain Name>

| Service | Path | Stack | Port | Owner | Purpose | Deps |
|---------|------|-------|------|-------|---------|------|
| `<name>` | `<path>` | <stack> | <port> | @team | <purpose> | <deps> |

## Adding a New Service

See `docs/service-template.md` for bootstrap checklist.
```

### Phase 2c: Generate `docs/service-template.md`

Adapt the checklist to the project's detected stacks:

```markdown
# New Service Checklist

## Backend (<detected stack>)

1. Copy `services/_template/` to `services/<name>/`
2. Update build config with service name and port
3. Create `services/<name>/CLAUDE.md` -- copy from any existing service CLAUDE.md
4. Add row to `docs/services.md` catalog
5. Add row to root `CLAUDE.md` service table (if in top domains)
6. Create config file with DB and port settings
7. Create `.env.example` with required vars
8. Register in `docker-compose.yml`

## Frontend (<detected stack>)

1. Scaffold new frontend with stack CLI
2. Create `frontends/<name>/CLAUDE.md` -- copy from any existing frontend CLAUDE.md
3. Add row to `docs/services.md` and root `CLAUDE.md`
4. Configure API proxy in dev config
5. Register in `docker-compose.yml`
```

### Phase 3: Generate Per-Service CLAUDE.md

Select the matching template based on detected stack.

#### 3a: Java / Spring Boot Backend

```markdown
# <Service Name> Service

<One-line purpose>

## Commands

| Command | Description |
|---------|-------------|
| `./gradlew :services:<name>:bootRun` | Run locally |
| `./gradlew :services:<name>:test` | Unit + integration tests |
| `./gradlew :services:<name>:build` | Build JAR |

## Architecture

```
src/main/java/com/<org>/<service>/
  controller/    # REST endpoints
  service/       # Business logic
  repository/    # JPA repositories
  domain/        # Entities & value objects
  dto/           # Request/Response DTOs
  config/        # Spring config & beans
src/main/resources/
  application.yml
  db/migration/  # Flyway migrations
```

## Key Files

- `<MainApp>.java` -- Spring Boot entry point
- `application.yml` -- profiles, datasource, ports
- `SecurityConfig.java` -- auth filter chain (if applicable)

## Dependencies

| Service | Protocol | Purpose |
|---------|----------|---------|
| `<dep>` | REST / Async | <purpose> |

## API & DB

- API contract: See `docs/api/<service>.yaml`
- DB: <database> (`<service>_db`) -- See `db/migration/`

## Env Vars

See `.env.example` -- key vars: `SPRING_DATASOURCE_URL`, `SERVER_PORT`

## Gotchas

- <service-specific quirk>
```

#### 3b: Node.js / Express Backend

```markdown
# <Service Name> Service

<One-line purpose>

## Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Dev server with hot reload |
| `npm test` | Run tests |
| `npm run build` | Production build |
| `npm start` | Start production server |

## Architecture

```
src/
  routes/        # Express route handlers
  controllers/   # Request/response logic
  services/      # Business logic
  models/        # Database models (Prisma/Sequelize/Mongoose)
  middleware/    # Auth, validation, error handling
  config/        # App configuration
  utils/         # Shared utilities
```

## Key Files

- `src/index.ts` -- Express app entry point
- `prisma/schema.prisma` -- Database schema (if Prisma)
- `.env.example` -- Required environment variables

## Dependencies

| Service | Protocol | Purpose |
|---------|----------|---------|
| `<dep>` | REST / Async | <purpose> |

## API & DB

- API contract: See `docs/api/<service>.yaml`
- DB: <database> -- See `prisma/schema.prisma` or `src/models/`

## Env Vars

See `.env.example` -- key vars: `DATABASE_URL`, `PORT`

## Gotchas

- <service-specific quirk>
```

#### 3c: Python / FastAPI Backend

```markdown
# <Service Name> Service

<One-line purpose>

## Commands

| Command | Description |
|---------|-------------|
| `uvicorn app.main:app --reload` | Dev server |
| `pytest` | Run tests |
| `alembic upgrade head` | Run DB migrations |

## Architecture

```
app/
  api/           # Route handlers
  core/          # Config, security, deps
  models/        # SQLAlchemy / Pydantic models
  schemas/       # Request/Response schemas
  services/      # Business logic
  repositories/  # Data access layer
```

## Key Files

- `app/main.py` -- FastAPI app entry point
- `alembic/` -- Database migrations
- `pyproject.toml` -- Dependencies and project config

## Dependencies

| Service | Protocol | Purpose |
|---------|----------|---------|
| `<dep>` | REST / Async | <purpose> |

## API & DB

- API contract: See `docs/api/<service>.yaml` or `/docs` (auto-generated)
- DB: <database> -- See `alembic/versions/`

## Env Vars

See `.env.example` -- key vars: `DATABASE_URL`, `PORT`

## Gotchas

- <service-specific quirk>
```

#### 3d: Go Backend

```markdown
# <Service Name> Service

<One-line purpose>

## Commands

| Command | Description |
|---------|-------------|
| `go run ./cmd/<name>` | Run locally |
| `go test ./...` | Run all tests |
| `go build -o bin/<name> ./cmd/<name>` | Build binary |

## Architecture

```
cmd/<name>/        # Entry point (main.go)
internal/
  handler/         # HTTP handlers
  service/         # Business logic
  repository/      # Data access
  model/           # Domain types
  middleware/      # HTTP middleware
config/            # Configuration
migrations/        # SQL migrations
```

## Key Files

- `cmd/<name>/main.go` -- Entry point
- `config/config.go` -- Environment-based config
- `go.mod` -- Dependencies

## Dependencies

| Service | Protocol | Purpose |
|---------|----------|---------|
| `<dep>` | REST / gRPC | <purpose> |

## API & DB

- API contract: See `docs/api/<service>.yaml`
- DB: <database> -- See `migrations/`

## Env Vars

See `.env.example` -- key vars: `DATABASE_URL`, `PORT`

## Gotchas

- <service-specific quirk>
```

#### 3e: Vue.js Frontend

```markdown
# <Frontend Name>

<One-line purpose>

## Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Dev server with HMR |
| `npm run build` | Production build |
| `npm run test:unit` | Vitest unit tests |
| `npm run lint` | ESLint + Prettier |

## Architecture

```
src/
  views/        # Page components (route-level)
  components/   # Reusable UI components
  composables/  # Vue composables (shared logic)
  stores/       # Pinia stores
  api/          # API client modules
  router/       # Vue Router config
  assets/       # Static assets, CSS
```

## Key Files

- `src/main.js` -- App entry, plugin registration
- `src/router/index.js` -- Route definitions
- `vite.config.js` -- Dev proxy to backend APIs

## API Dependencies

| Backend Service | Base Path | Purpose |
|----------------|-----------|---------|
| `<service>` | `/api/<path>` | <purpose> |

## Env Vars

See `.env.example` -- key var: `VITE_API_BASE_URL`

## Gotchas

- <frontend-specific quirk>
```

#### 3f: React / Next.js Frontend

```markdown
# <Frontend Name>

<One-line purpose>

## Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Dev server with HMR |
| `npm run build` | Production build |
| `npm test` | Jest / Vitest tests |
| `npm run lint` | ESLint |

## Architecture

```
src/
  app/          # Next.js App Router pages (or pages/ for Pages Router)
  components/   # Reusable UI components
  hooks/        # Custom React hooks
  lib/          # Utilities, API clients
  stores/       # State management (Zustand / Redux)
  styles/       # Global styles, Tailwind config
```

## Key Files

- `src/app/layout.tsx` -- Root layout
- `next.config.js` -- Rewrites, env, redirects
- `tailwind.config.js` -- Design tokens (if Tailwind)

## API Dependencies

| Backend Service | Base Path | Purpose |
|----------------|-----------|---------|
| `<service>` | `/api/<path>` | <purpose> |

## Env Vars

See `.env.example` -- key var: `NEXT_PUBLIC_API_URL`

## Gotchas

- <frontend-specific quirk>
```

**Per-service rules:**
- Max 80 lines per service CLAUDE.md
- All doc references use `See` pointers, never inline content
- Dependencies table: only direct service-to-service deps
- Env vars: only list critical ones, point to `.env.example` for rest
- Architecture tree: max 10 lines, show top-level dirs only

### Phase 4: Validation

After generation, verify:

```
[ ] Root CLAUDE.md < 100 lines (count with `wc -l`)
[ ] Each service CLAUDE.md < 80 lines
[ ] No code snippets inlined (only file pointers)
[ ] All `See` targets actually exist
[ ] Service table matches actual directory structure
[ ] Commands are copy-paste runnable
[ ] No generic advice (only project-specific content)
[ ] `See` used everywhere, never `@`
[ ] Stack-specific commands match detected build tools
```

Report line counts:
```bash
wc -l ./CLAUDE.md ./services/*/CLAUDE.md ./frontends/*/CLAUDE.md 2>/dev/null
```

### Phase 5: Gap Analysis

Check for referenced docs that don't exist yet:

```bash
grep -roh 'See `[^`]*`' ./CLAUDE.md ./services/*/CLAUDE.md ./frontends/*/CLAUDE.md 2>/dev/null | \
  sed "s/See \`//;s/\`//" | while read f; do
    [ ! -e "$f" ] && echo "MISSING: $f"
  done
```

For each missing reference, create a one-line stub file so the pointer isn't broken:

```bash
echo "# <Title> -- TODO: Add content" > <missing-file>
```

## Anti-Patterns (NEVER do these)

- Inline code examples in CLAUDE.md -- use `See` pointers
- Use `@` references -- loads at startup, defeats context savings
- Duplicate info across root and service CLAUDE.md files
- Generic best practices ("use meaningful variable names")
- Full API endpoint listings -- point to OpenAPI spec
- Environment variable values -- point to `.env.example`
- Config file contents -- point to the actual file
- Architecture diagrams as ASCII art > 10 lines -- point to docs
- Root CLAUDE.md > 100 lines
- Stack-specific details in root -- belong in per-service files

## When to Update

- New service added -> add row to root service table + create service CLAUDE.md
- Service removed -> remove row + delete service CLAUDE.md
- API contract changed -> verify `See` pointers still valid
- Tech stack changed -> update stack column in service table
- New domain -> add domain group heading in root

## Related Skills

- **coding-standards** -- Code conventions referenced from per-service CLAUDE.md files
- **backend-patterns** -- API design and architecture patterns for backend services
- **frontend-patterns** -- UI component and state management patterns for frontends
- **claude-md-improver** -- Audit and improve existing CLAUDE.md files (complementary)

---
> Source: [cskwork/msa-claude-md](https://github.com/cskwork/msa-claude-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
