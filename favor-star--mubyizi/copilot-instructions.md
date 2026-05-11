## mubyizi

> Workforce management platform for tracking daily-rate workers across multiple organizations and worksites. Manages attendance, payments, expenses, and role-based access for industries like construction, agriculture, events, and logistics.

# Mubyizi

Workforce management platform for tracking daily-rate workers across multiple organizations and worksites. Manages attendance, payments, expenses, and role-based access for industries like construction, agriculture, events, and logistics.

## Tech Stack

| Layer | Tech |
|---|---|
| Monorepo | Turborepo + pnpm workspaces |
| API | Hono.js + Node.js (ESM) |
| Frontend | Next.js 16 + React 19 |
| Database | PostgreSQL + Prisma ORM |
| Auth | better-auth (email/password) |
| Validation | Zod v4 |
| UI Components | shadcn/ui + Tailwind CSS v4 |
| API Docs | hono-openapi + Scalar UI |

## Key Directories

```
apps/api/src/
  main.ts              # Hono app entry, middleware setup, error handler
  routes/              # Route handlers (one file per domain)
  schemas/             # Zod schemas for validation and OpenAPI
  docs/                # OpenAPI route documentation descriptors
  middleware/          # auth.middleware.ts, roles.middleware.ts
  lib/                 # auth.ts (better-auth), prisma-client.ts
  helpers/             # api, validation, pagination, org, workplace helpers
  _constants/          # Role weights, API base path, OpenAPI tags
  _types/              # HonoInstanceContext, shared types

apps/api/prisma/
  schema.prisma        # better-auth base models (User, Session, Account)
  models/              # Extended domain models (split by domain)

apps/web/
  app/                 # Next.js App Router pages
  components/          # React components (providers.tsx for themes)

packages/ui/src/       # Shared shadcn/ui component library (@workspace/ui)
```

## Build & Dev Commands

```bash
# From repo root
pnpm dev              # Start all apps (API on :3001, Web on :3000)
pnpm build            # Build all apps
pnpm lint             # Lint all packages

# From apps/api/
pnpm dev              # API with hot reload (tsx --watch)
pnpm build            # tsc → dist/
pnpm start            # Run compiled API (production)

# Database
cd apps/api && npx prisma migrate dev   # Apply migrations
cd apps/api && npx prisma generate      # Regenerate client
cd apps/api && npx prisma studio        # GUI browser
```

## API Base Path

All API routes: `/api/v1`
Docs UI: `http://localhost:3001/api/v1/docs/`
OpenAPI JSON: `http://localhost:3001/api/v1/docs/openapi.json`

## Role Hierarchy

Three independent RBAC levels (see `_constants/role-weights.constants.ts`):
- **System**: `SUPERADMIN > SUPPORT > USER` — SUPERADMIN bypasses all org/workplace checks
- **Org**: `OWNER > ADMIN > MANAGER > MEMBER > VIEWER`
- **Workplace**: `WORKPLACE_MANAGER > SUPERVISOR > WORKER > VISITOR`

## Environment Variables

API (`apps/api/.env`): `PORT`, `DATABASE_URL`, `BETTER_AUTH_BASE_URL`, `FRONTEND_URL`, `SERVER_URL`
Web (`apps/web/.env`): `NEXT_PUBLIC_API_URL`

## Additional Documentation

Check these files when working on related topics:

| Topic | File |
|---|---|
| Architecture, patterns, conventions | `.claude/docs/architectural_patterns.md` |
| RBAC roles — what each role can/cannot do | `.claude/docs/roles-reference.md` |

---
> Source: [Favor-star/mubyizi](https://github.com/Favor-star/mubyizi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
