## beguardit-cli

> BeGuardit CLI is a terminal-first cybersecurity and AI security assessment platform.

# CLAUDE.md — BeGuardit Development Context

## Project Overview
BeGuardit CLI is a terminal-first cybersecurity and AI security assessment platform.
Monorepo with 6 components: CLI (Node.js 20), API (FastAPI/Python 3.12+), Worker (ARQ/Python),
Portal (React 18 + Vite + TypeScript + Tailwind), Public Site (Astro 4), Release Pipeline (GitHub Actions).

## Architecture Reference
- Full spec: docs/architecture.md and ARCH-002-2026-03-17 PDF
- ADRs: docs/adr/ADR-001 through ADR-010
- Database: PostgreSQL 16, shared-schema multi-tenancy, tenant_id on every table
- Queue: Redis 7 via ARQ
- Auth: Session-based (HttpOnly cookies), argon2id password hashing, RBAC (admin/operator/viewer)

## Code Conventions

### Python (api/, worker/)
- Python 3.12+ with full type hints (mypy compatible)
- FastAPI domain module pattern: router.py, schemas.py, models.py, service.py, dependencies.py, exceptions.py
- SQLAlchemy 2.0 mapped_column style with AsyncSession
- Pydantic v2 for all request/response schemas
- structlog for all logging (JSON structured)
- All DB queries MUST include tenant_id in WHERE clause
- Alembic for migrations in api/alembic/versions/
- Use `from __future__ import annotations` in every file
- Linting: ruff

### JavaScript/TypeScript (cli/, portal/)
- CLI: Node.js 20, Commander.js, Inquirer.js, Vitest for tests
- Portal: React 18, TypeScript strict (no `any`), Vite 5, Tailwind CSS 3, TanStack Query
- ESLint + Prettier
- No external UI component libraries — custom Tailwind components only

### Database
- PostgreSQL 16. UUID primary keys via gen_random_uuid()
- Every tenant-scoped table has tenant_id UUID NOT NULL with FK to tenants(id)
- Indexes on tenant_id + created_at DESC for list queries
- Alembic migrations: descriptive names, one migration per feature

### API Patterns
- All endpoints under /api/v1/
- Standard error schema: { "error": { "code": "string", "message": "string", "detail": object|null, "correlation_id": "string" } }
- Pagination: offset/limit with { items, total, offset, limit } response
- Auth via session cookie, tenant from session, role checked via require_role() dependency

### File Headers
Every source file starts with a comment block referencing the architecture section:
```
# <Module> — <purpose>
# Source: ARCH-002-2026-03-17, Section <N>
```

### Testing
- API: pytest + httpx async client, 80% coverage minimum
- CLI: Vitest
- Portal: Vitest + React Testing Library
- E2E: Playwright

### Infrastructure
- Docker Compose for dev (infra/docker-compose.yml)
- Kubernetes for prod (infra/k8s/)
- All secrets via environment variables, never hardcoded
- GitHub Actions CI/CD (.github/workflows/)

## Current Task Context
Working on architecture gap fixes from ARCH-002 v1 review. Each fix is an isolated unit of work.
Do NOT modify existing working code unless the fix specifically requires it.
Every database change needs an Alembic migration.
Every new endpoint needs a test.
Every new worker job needs idempotency checks.

---
> Source: [benchbrex-USA/BEGUARDIT-CLI](https://github.com/benchbrex-USA/BEGUARDIT-CLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
