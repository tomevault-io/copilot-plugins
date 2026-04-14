## minimal-tanstack-saas-starter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm install                  # Install dependencies
pnpm dev                      # Dev server on port 3000
pnpm build                    # Production build
pnpm lint                     # ESLint check
pnpm format                   # Prettier check
pnpm check                    # Prettier fix + ESLint fix
pnpm test                     # Run all tests (Vitest)

# Database (Drizzle + PostgreSQL, run from apps/web)
pnpm db:generate              # Generate migrations from schema
pnpm db:migrate               # Run migrations
pnpm db:push                  # Push schema directly (dev)
pnpm db:studio                # Open Drizzle Studio
```

Run a single test: `cd apps/web && pnpm vitest run src/path/to/test.test.ts`

## Architecture

**Monorepo** using pnpm workspaces + Turborepo. Currently one app at `apps/web`.

### Tech Stack
- **Framework**: TanStack Start (full-stack React with SSR via Nitro/Vite)
- **Routing**: TanStack Router with file-based routes (`apps/web/src/routes/`)
- **Data fetching**: TanStack Query + TanStack Router loaders
- **Auth**: Better Auth with email/password + organization plugin (`src/lib/auth.ts`)
- **Database**: PostgreSQL via Drizzle ORM (`src/db/schema.ts`, `src/db/index.ts`)
- **UI**: Tailwind CSS v4 + shadcn/ui components (`src/components/ui/`)
- **Forms**: TanStack Form
- **Tables**: TanStack Table
- **Validation**: Zod v4

### Key Patterns

**Path alias**: `#/*` maps to `apps/web/src/*` (configured in package.json `imports`). Use `@/` in code (resolved by vite-tsconfig-paths).

**Route structure**: `__root.tsx` wraps everything with providers (QueryClient, Theme, Toaster). `_authed.tsx` is a layout route that redirects unauthenticated users and renders sidebar + header.

**Auth flow**:
- Server-side: `getSession`, `requireSession`, `requireOrgSession` server functions in `src/lib/auth-server.ts`
- Client-side: `authClient` exports from `src/lib/auth-client.ts` (`useSession`, `useActiveOrganization`, `signIn`, `signUp`, `signOut`)
- Session context is injected into the router context at `__root.tsx` beforeLoad

**Organizations**: Every new user gets an auto-created org. The `_authed` layout redirects users whose org still has the default name (`'s Org`) to `/onboarding`.

**Database schema**: Drizzle schema uses camelCase column names to match Better Auth's Kysely adapter. Tables: `user`, `session`, `account`, `verification`, `organization`, `member`, `invitation`, `team`, `teamMember`.

**Env vars**: `DATABASE_URL`, `BETTER_AUTH_URL`, `BETTER_AUTH_SECRET` in `.env.local`.

## Code Style

- Prettier: no semicolons, single quotes, trailing commas
- ESLint: TanStack config with relaxed rules (no import ordering enforcement)
- `verbatimModuleSyntax` enabled — use `import type` for type-only imports
- Add shadcn components via: `pnpm dlx shadcn@latest add <component>`

## TanStack Skills

Use these slash-command skills for idiomatic TanStack patterns:
- `/tanstack-start` — server functions, middleware, SSR, auth, deployment
- `/tanstack-router` — type-safe routing, data loading, search params, navigation
- `/tanstack-query` — data fetching, caching, mutations, server state
- `/tanstack-integration` — coordinating Query + Router + Start for full-stack data flow

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/markoinla)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/markoinla)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
