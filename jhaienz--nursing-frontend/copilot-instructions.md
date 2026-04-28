## nursing-frontend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
pnpm install

# Development (all packages)
pnpm dev

# Run only the web app
pnpm --filter web dev

# Build
pnpm build

# Type-check
pnpm typecheck

# Lint
pnpm lint

# Format
pnpm format
```

## Architecture

**Turborepo monorepo** with two packages:

- `apps/web` — React 19 + Vite SPA (the main app)
- `packages/ui` — Shared shadcn/ui component library

The web app talks to a companion NestJS REST API (default `http://localhost:3000`, overridable via `VITE_API_URL`).

### Key libraries

| Library | Purpose |
|---------|---------|
| React Router v7 | Client-side routing |
| TanStack Query v5 | Server state / data fetching |
| Zustand v5 | Client state (auth session) |
| React Hook Form + Zod | Form validation |
| Axios | HTTP client |
| Tailwind CSS v4 | Styling |
| lucide-react | Icons |

### Route structure

Two route groups defined in [apps/web/src/routes.tsx](apps/web/src/routes.tsx):

- `/admin/*` — JWT-protected admin panel (login, dashboard, borrowers, items, loans, penalties). Wrapped in `PrivateRoute` which redirects to `/admin/login` if unauthenticated.
- `/borrower/*` — Public borrower portal (browse items, submit loan requests, view own loans). No auth required.

### State management

**Auth** is a Zustand store ([apps/web/src/lib/auth.ts](apps/web/src/lib/auth.ts)) that persists `auth_token` and `auth_user` to `localStorage`. `restoreSession()` is called once on app mount.

**Server state** lives entirely in TanStack Query hooks under [apps/web/src/lib/hooks/](apps/web/src/lib/hooks/). Each domain (auth, borrowers, items, loans, penalties) has its own hook file. Mutations call `queryClient.invalidateQueries` on success to keep the cache fresh.

### API client

`apiClient` ([apps/web/src/lib/api.ts](apps/web/src/lib/api.ts)) is a pre-configured Axios instance:
- Auto-injects `Authorization: Bearer <token>` from `localStorage` on every request.
- On 401: only redirects to `/admin/login` if the request itself carried a token — this prevents borrower PIN-error responses from triggering a redirect.

### Shared UI components

Components are added via shadcn/ui CLI and land in `packages/ui/src/components/`. Import them in the app as:

```tsx
import { Button } from "@workspace/ui/components/button"
```

Global CSS is exported from `packages/ui/src/styles/globals.css` and imported in the app as `@workspace/ui/globals.css`.

### Types

All shared API response types are defined in [apps/web/src/lib/types.ts](apps/web/src/lib/types.ts). Field names use camelCase to match the Prisma/API JSON output (e.g. `isBlocked`, `isReturnable`, `dueDate`).

## Adding shadcn/ui components

Run from the repo root, targeting `apps/web`:

```bash
pnpm dlx shadcn@latest add <component> -c apps/web
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jhaienz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
