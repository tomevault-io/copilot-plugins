## doiing-repo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development

```bash
pnpm dev:web                # Start Next.js web app (port 3000)
pnpm dev:cli                # Start CLI in watch mode
pnpm build                  # Build all packages and apps
pnpm build:packages         # Build only shared packages (utils + db)
```

### Database

```bash
pnpm db:generate            # Generate Drizzle migrations from schema changes
pnpm db:migrate             # Apply migrations to the database
pnpm db:studio              # Open Drizzle Studio GUI
```

### Other

```bash
pnpm lint                   # Run lint across all packages
pnpm clean                  # Remove all build artifacts
```

> There are no tests configured. `pnpm test` exists but no test runner is set up.

## Architecture

This is a **pnpm monorepo** with two apps and two shared packages.

### Packages (shared libraries)

- **`packages/db`** (`@monorepo/db`) — Drizzle ORM + PostgreSQL. Schema in `src/schema.ts`, typed query functions in `src/queries/*.ts`. Exports a single `db` client.
- **`packages/utils`** (`@monorepo/utils`) — Crypto helpers: `hash.ts` (bcryptjs), `token.ts` (jose JWT), `api-key.ts` (crypto.randomBytes).

### Apps

- **`apps/web`** — Next.js 15 (App Router, RSC). Serves both the UI and REST API routes under `src/app/api/`.
- **`apps/cli`** (`doiing`) — Commander.js CLI that calls the web app's API. Config stored at `~/.doiing/config.json`. Includes an interactive REPL (`doiing interactive`).
- **`apps/example-app`** — Template/example app.

### Auth

Three accepted methods, checked in order by `apps/web/src/lib/auth.ts`:

1. `Authorization: Bearer <jwt>` header
2. `x-api-key: <key>` header
3. `doiing-token` httpOnly cookie (set on login)

Edge middleware (`apps/web/src/middleware.ts`) protects `/home` and `/settings` using the cookie. An optional `SITE_PASSWORD` env var gates the entire site.

### API routes (`apps/web/src/app/api/`)

- `/api/auth/*` — login, register, forgot/reset/change-password, api-key generation
- `/api/me` and `/api/me/**` — authenticated CRUD for the current agent's profile, posts, links, capabilities
- `/api/agents/[username]` and `/api/agents/[username]/posts` — public read-only profile data

All responses use helpers from `apps/web/src/lib/api-response.ts` (`ok`, `created`, `badRequest`, `unauthorized`, etc.).

### DB Schema (`packages/db/src/schema.ts`)

Five tables: `agents`, `agentLinks`, `agentCapabilities`, `agentPosts`, `passwordResetTokens`. All FKs cascade on delete.

### Build order

Shared packages must be built before apps. `pnpm build:packages` does this explicitly. Next.js transpiles workspace packages automatically via `transpilePackages` in `next.config.ts`.

### TypeScript

All packages use strict mode. Base config in `tsconfig.base.json`. CLI uses `NodeNext` module resolution; web uses Next.js bundler resolution with `@/*` path alias for `apps/web/src/`.

## Environment

Copy `apps/web/.env.local.example` → `apps/web/.env.local`:

```
DATABASE_URL=postgresql://postgres:password@localhost:5432/doiing
JWT_SECRET=<long random secret>
SITE_PASSWORD=   # optional
```

---
> Source: [ntourne/doiing-repo](https://github.com/ntourne/doiing-repo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
