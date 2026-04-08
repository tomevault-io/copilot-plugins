## tanstack-playground

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vite + React + TypeScript playground focused on TanStack Router and TanStack Query. UI library is Mantine v8. Deployed to Cloudflare Workers via Wrangler.

The backend lives in a sibling directory ending with `-backend`. When implementation depends on backend behavior or API contracts, inspect that sibling project first.

## Commands

- `pnpm dev` — start dev server on port 3000
- `pnpm test` — run Vitest (jsdom, `src/**/*.test.{ts,tsx}`)
- `pnpm test -- src/path/to/file.test.ts` — run a single test file
- `pnpm lint` — Biome check
- `pnpm lint:fix` — Biome auto-fix
- `pnpm typecheck` — TypeScript check via `tsgo --noEmit`
- `pnpm build` — production build
- `pnpm preview` — build + serve locally with Wrangler

Pre-commit hook (lefthook) runs `pnpm lint` and `pnpm typecheck` in parallel. Both must pass before commit.

## Architecture

### Routing

TanStack Router with file-based routing (`src/routes/`). Route tree is auto-generated in `src/routeTree.gen.ts` (do not hand-edit). The router plugin uses `autoCodeSplitting: true`.

- `src/routes/__root.tsx` — root layout with MantineProvider, AppShell, auth state, devtools
- `src/routes/_authenticated.tsx` — layout route that guards child routes via `beforeLoad` session check; redirects to `/` if unauthenticated
- `src/routes/_authenticated/tickets/` — ticket CRUD routes (`index`, `new`, `$ticketId/index`, `$ticketId/edit`)

Route components are thin wrappers in `src/routes/` that delegate to feature-level route components in `src/features/*/routes/`.

### Feature Modules (`src/features/`)

Each feature follows this structure:
- `api.ts` — fetch functions using the shared HTTP client
- `schema/` — Zod schemas for validation and type inference
- `hooks/` — TanStack Query hooks (queries + mutations)
- `components/` — React components organized by concern (list, detail, forms, dialogs, comments, feedback, layout)
- `routes/` — route-level components that compose hooks + components
- `queryKeys.ts` — query key factory pattern for cache management

### Shared (`src/shared/`)

- `api/http.ts` — HTTP client utilities: `createApiUrl`, `ensureSuccess`, error classes (`HttpError`, `UnauthorizedError`)
- `config/env.ts` — typed environment config (`VITE_API_BASE_URL`)
- `ui/toast.tsx` — toast notification system
- `utils/` — utility functions (date, searchParam, url)

### Path Aliases

`@/*` maps to `./src/*`. All imports use `@/` prefix.

### Testing

- Vitest + Testing Library + jsdom
- MSW for API mocking (`src/mocks/handlers.ts`, `src/mocks/node.ts`)
- Test fixtures in `src/test/fixtures/`
- Tests are colocated with source files as `*.test.{ts,tsx}`

### Key Patterns

- **Query key factory**: `ticketsQueryKey.list(filter)`, `ticketsQueryKey.detail(id)` — used for targeted cache invalidation
- **Router context**: `QueryClient` is passed via router context, enabling `beforeLoad` data fetching
- **Form handling**: react-hook-form + @hookform/resolvers + Zod schemas
- **Auth flow**: session check in `_authenticated` layout route; `AuthRedirectController` handles redirect logic

## Coding Conventions

- Biome is the formatter/linter (not ESLint/Prettier). See `biome.json` for config.
- Single quotes, semicolons, trailing commas, 2-space indent, max line width 100
- Prefer named React imports (no default `React` import)
- Commit style: `feat:`, `fix:`, `refactor:`, `chore:` with concise scopes
- Environment variable: `VITE_API_BASE_URL` (dev default: `http://localhost:7080`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/voice0726)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/voice0726)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
