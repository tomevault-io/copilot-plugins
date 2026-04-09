## burntop

> > Guidelines for AI coding agents working in this repository.

# AGENTS.md - burntop.dev

> Guidelines for AI coding agents working in this repository.

## Project Overview

Gamified AI usage tracking platform. Monorepo: backend (FastAPI), frontend (TanStack Start), CLI (bun).

## Agent Instructions

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

### Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --status in_progress  # Claim work
bd close <id>         # Complete work
bd sync               # Sync with git
```

## Build & Development Commands

```bash
bun install                     # Install dependencies
bun dev                        # Start all packages
bun build                       # Build all packages
bun lint / bun lint:fix         # ESLint (all packages)
bun format                      # Prettier formatting
bun typecheck                   # TypeScript type checking

# Single file tests (CRITICAL)
cd packages/frontend && npx vitest run path/to/file.test.ts  # Frontend
cd packages/backend && uv run pytest tests/unit/test_file.py    # Backend

# Backend dev
cd packages/backend && uv run uvicorn src.app.main:app --reload
cd packages/backend && uv run ruff check src/    # Lint
cd packages/backend && uv run ruff format src/   # Format

# Database
cd packages/backend && uv run alembic upgrade head    # Run migrations
cd packages/backend && uv run alembic revision --autogenerate -m "desc"
```

## Code Style Guidelines

### TypeScript/React

- Strict mode: no `any`, no `@ts-ignore`
- Explicit return types on exports
- Prefer `interface` for objects, `type` for unions
- Use `import type` for type-only imports
- React 19: hooks, functional components only

### Python (Backend)

- Python 3.12+, type hints required
- Async SQLAlchemy 2.0 with `Mapped[]`
- Use `Decimal` for financial calculations
- Use `datetime.UTC` instead of `timezone.utc`

### Import Order (TypeScript)

```typescript
// 1. React/framework, 2. Third-party, 3. @/, 4. Relative, 5. type
import { createFileRoute } from '@tanstack/react-router';
import { z } from 'zod';
import { db } from '@/lib/db';
import { StatsCard } from './stats-card';
import type { User } from '@/types';
```

### Naming Conventions

- Files: `kebab-case.tsx` (components), `kebab-case.ts` (utils)
- Components: `PascalCase`, Functions: `camelCase`, Constants: `SCREAMING_SNAKE_CASE`
- DB tables: `snake_case`, Routes: `/kebab-case`

## Skills for Common Tasks

### Backend (FastAPI)

- `/fastapi-entity` - New entity/model
- `/fastapi-auth` - Authentication
- `/fastapi-database-setup` - Database config
- `/fastapi-core-*` - Models, schemas, repository, service, app factory

### Frontend (TanStack Start)

- `/tanstack-start-routing` - New routes
- `/tanstack-react-query-patterns` - Data fetching
- `/tanstack-start-server-functions` - Server functions
- `/tanstack-shadcn-*` - Components, forms, data tables

### Visual/UI Changes

Delegate styling, layout, animations to `frontend-ui-ux-engineer` agent

### Other

- `/tanstack-client-auth` - Token-based auth
- `/tanstack-client-api-layer` - API client setup
- `/fastapi-testing` - Backend testing

## Critical Rules

- **No `any` types** (TS/Python) without justification
- **No `console.log`** in production code
- **Frontend client-side only** - all DB ops via FastAPI backend
- **Route files must NOT import `@/lib/db` directly** - use server functions
- **CLI requires bun runtime** - uses `bun:sqlite`
- **Dark mode only** - Base: `--bg-base: #0A0A0A`, Accent: `--ember-500: #FF6B00`

## Project-Specific Patterns

### Frontend Components

```typescript
import { cn } from '@/lib/utils';
import { Card } from '@/components/ui/card';

export function StatsCard({ className }: { className?: string }) {
  return <Card className={cn('bg-bg-elevated border-border-default', className)} />;
}
```

### API Integration

```typescript
// Regenerate after backend changes: cd packages/frontend && bun run generate:api
import { useGetUserProfileApiV1UsersUsernameGet } from '@/api/users';
const { data } = useGetUserProfileApiV1UsersUsernameGet({ username: 'johndoe' });
```

### Backend Architecture (3-Layer)

```
packages/backend/src/app/{entity}/
├── models.py       # SQLAlchemy ORM
├── schemas.py      # Pydantic schemas
├── repository.py   # Data access
├── service.py      # Business logic
├── router.py       # FastAPI routes
└── dependencies.py # DI functions
```

**Pattern**: Routers → Services → Repositories. Never skip layers.

## Local Database Setup

```bash
docker compose -f docker-compose.db.yml up -d          # Start
cd packages/backend && uv run alembic upgrade head      # Migrate
docker compose -f docker-compose.db.yml down          # Stop
```

**Connection**: `postgresql://burntop:burntop_dev_password@localhost:5433/burntop`

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd sync
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**

- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/agusmdev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
