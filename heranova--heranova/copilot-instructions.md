## heranova

> Contact management system for a CBD shop. Handles professional contact registration with SIRET validation, welcome emails via Resend, and an admin backoffice for managing contacts and email logs.

# Heranova

Contact management system for a CBD shop. Handles professional contact registration with SIRET validation, welcome emails via Resend, and an admin backoffice for managing contacts and email logs.

## Tech Stack

- **Runtime**: Bun 1.3.7
- **Monorepo**: Turborepo with Bun workspaces
- **Frontend** (`apps/frontend`): SvelteKit 2, Svelte 5, Tailwind CSS 4, shadcn-svelte, adapter-node
- **Backoffice** (`apps/backoffice`): SvelteKit 2, Svelte 5, Tailwind CSS 4, shadcn-svelte, Chart.js, Iconify
- **Backend** (`apps/backend`): Hono on Bun, Zod env validation, rate limiting, CSRF/CORS/secure headers
- **Database**: PostgreSQL via Prisma 6 (client generated to `packages/database/generated/client`)
- **Email**: Resend
- **CI**: GitHub Actions (lint, type-check, test, build, Docker build) on `main` and `dev` branches
- **Deploy**: Railway (each app has its own Dockerfile, all built from monorepo root)

## Commands

All commands run from the monorepo root:

| Command | What it does |
|---|---|
| `bun install` | Install all dependencies |
| `bun run dev` | Start all apps in dev mode (Turbo) |
| `bun run build` | Build all apps |
| `bun run lint` | Lint all apps (ESLint) |
| `bun run type-check` | Type-check all apps |
| `bun run test` | Run all tests |
| `bun run clean` | Remove build artifacts and node_modules |

Add dependencies to a specific workspace:

```
bun run add:frontend <package>
bun run add:backend <package>
bun run add:backoffice <package>
bun run add:types <package>
bun run add:validator <package>
```

Database commands (from `packages/database`):

```
bun run migrate:dev    # Create and apply dev migration
bun run migrate:deploy # Apply migrations in production
```

## Project Structure

```
apps/
  frontend/        SvelteKit public-facing site (contact form, "bientot-disponible" page)
  backoffice/      SvelteKit admin panel (contacts, email logs, auth via token)
  backend/         Hono REST API
    src/
      index.ts          Entry point (Bun.serve)
      router.ts         Hono app with middleware stack
      lib/              Utilities (env validation, mail service, response helpers)
      middleware/        Rate limiting, auth
      routes/
        contacts/       Contact CRUD (route, controller, service)
        backoffice/     Backoffice endpoints
        checkout/       Checkout flow
        logs/           Email log endpoints
packages/
  database/        Prisma schema, migrations, generated client
  types/           Shared TypeScript types (depends on database)
  validator/       Zod schemas for contacts and pagination
```

## Environment Variables

Copy `.env.example` to `.env` at the root and in each app that needs it. Required backend vars:

- `DATABASE_URL` -- PostgreSQL connection string
- `RESEND_API_KEY` -- Resend email API key
- `RESEND_FROM_EMAIL` -- Sender email address
- `BACKOFFICE_TOKEN` -- Admin auth token (min 32 chars)
- `PORT` -- Server port (default 8080)
- `ALLOWED_ORIGINS` -- Comma-separated CORS origins (optional)

## Database

Two models in `packages/database/prisma/schema.prisma`:

- `Contact` -- Professional contacts with SIRET, email (unique), company info
- `EmailLog` -- Tracks sent emails with status

After schema changes: `cd packages/database && bun run migrate:dev`

## Conventions

- Internal packages use `workspace:*` protocol
- Backend follows route/controller/service pattern per resource
- Backend env is validated at startup via Zod (`apps/backend/src/lib/env.ts`) -- the server won't start with missing or invalid vars
- SvelteKit apps use adapter-node for Railway deployment
- Dockerfiles are per-app but require the full monorepo context (root directory must be `/` in Railway)
- TypeScript strict mode enabled globally
- No tests exist yet -- all test scripts are stubs

---
> Source: [Heranova/Heranova](https://github.com/Heranova/Heranova) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
