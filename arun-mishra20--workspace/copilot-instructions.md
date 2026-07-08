## workspace

> Focus on implementing features, not tests.

# Copilot Instructions

Focus on implementing features, not tests.

## Stack

- **Backend:** NestJS 11, Drizzle ORM, Passport JWT, Pino
- **Frontend:** Vite, React 19, React Router, TanStack Query, React Hook Form
- **Infra:** Turborepo, pnpm workspaces, TypeScript 5.9, PostgreSQL

## Monorepo Structure

```
apps/api/          # NestJS (port 3000)
apps/web/          # Vite + React (port 8000)
packages/database/ # Drizzle schemas & migrations
packages/ui/       # Shared shadcn/ui + Tailwind components
```

## Backend: DDD Module Layout

```
modules/{feature}/
  application/    # Services (use cases), ports (interfaces), DTOs
  domain/         # Entities, value objects
  infrastructure/ # Drizzle repositories, 3rd-party adapters, Passport strategies
  presentation/   # Controllers, guards, decorators
```

Auth module uses 3+2+1 design: multiple `auth_identities` per user, sessions in `auth_sessions`, single `users` aggregate root.

## Frontend Layout

```
src/pages/        # Route page components
src/features/     # Feature-specific logic, hooks, utils
src/components/   # Reusable UI (shadcn/ui + Tailwind)
src/lib/          # Shared utilities
```

Use TanStack Query for server state, React Hook Form for forms. Generate API types: `cd apps/web && pnpm generate:api` (requires API running).

## Conventions

- Files: `*.controller.ts`, `*.service.ts`, `*.repository.ts`, `*.module.ts`, `*.guard.ts`
- Tests: `*.spec.ts` (unit), `*.e2e-spec.ts` (E2E)
- Imports: `@/` aliases defined in `packages/typescript-config/tsconfig.base.json`

## Common Tasks

**New NestJS module:**

1. Create `apps/api/src/modules/{feature}/` with `application/`, `domain/`, `infrastructure/`, `presentation/`
2. Define ports → domain logic → repositories → controller → `{feature}.module.ts`
3. Import in `app.module.ts`

**New DB schema:**

1. Add schema to `packages/database/src/schemas/{entity}.ts`
2. Export from `schemas/index.ts`, add relations to `relations.ts`
3. Run `pnpm db:generate` then `pnpm db:push`

**New frontend route:**

1. Create `apps/web/src/pages/{route}.tsx`
2. Register the route in the React Router config
3. Use TanStack Query for data fetching, Tailwind for styles

## Key Commands

```bash
pnpm start:dev          # Start api + web
pnpm lint / lint:fix    # ESLint
pnpm check-types        # TypeScript
pnpm build              # Full build (Turborepo)

# Database (from packages/database/)
pnpm db:push / db:generate / db:studio
```

## Library & Utility Conventions

### Validation — always Zod

- Use Zod for ALL validation: API inputs, env vars, form schemas, data parsing
- Never use manual `if` checks or `class-validator` alone for data shape validation
- Define schemas in a `*.schema.ts` file colocated with the feature
- Prefer `z.infer<typeof Schema>` for deriving TypeScript types

### Date handling — always date-fns

- Never use `new Date()` arithmetic, `.toLocaleDateString()`, or manual date math
- Use date-fns functions: `format`, `parseISO`, `differenceInDays`, `addDays`, etc.
- Import only what you need: `import { format } from 'date-fns'`

### Utility functions — always lodash

- Prefer lodash over hand-rolled array/object manipulation
- Use `_.groupBy`, `_.uniqBy`, `_.omit`, `_.pick`, `_.merge`, `_.cloneDeep`, etc.
- Import per-method to keep bundles small: `import groupBy from 'lodash/groupBy'`
- Never reimplement: deduplication, deep clone, grouping, flattening, sorting

### Shared utilities — check before writing

- Before writing any helper, check these locations first:
  - `apps/api/src/lib/` — backend utilities
  - `apps/web/src/lib/` — frontend utilities
  - `packages/ui/src/` — shared UI helpers
- If a utility already exists, import it. Do not duplicate it.
- If you create a new generic utility, add it to the appropriate `lib/` folder, not inline

## Error Handling & Auth

- Global filters: `AllExceptionsFilter`, `ProblemDetailsFilter` (RFC 7807), `ThrottlerExceptionFilter`
- JWT: Bearer tokens for access, httpOnly cookies for refresh
- Swagger: `/api/docs` | OpenAPI export: `/openapi.yaml`

---
> Source: [arun-mishra20/workspace](https://github.com/arun-mishra20/workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
