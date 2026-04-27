## cafeina-frontend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## AWS Profile

This project uses the **dive** AWS profile. Before running any AWS commands (ECS deployments, etc.), switch to it:

```bash
aws-set-default dive
```

## Build & Development Commands

```bash
pnpm install          # Install dependencies (uses frozen lockfile)
pnpm dev              # Start dev server on http://localhost:5173
pnpm build            # TypeScript check + Vite production build
pnpm preview          # Preview production build on http://localhost:4173
pnpm lint             # Run ESLint
```

No test framework is configured yet.

## Architecture

React 19 + Vite 7 + TypeScript SPA for a coffee import/distribution ERP. Deployed via Docker/Nginx to AWS ECS.

### Tech Stack

- **Routing**: React Router v7 (nested routes)
- **Server State**: TanStack React Query (5-min stale time, custom retry logic skipping 4xx)
- **HTTP**: Axios instance with interceptors (`lib/api/axios.ts`), base URL from `VITE_API_BASE_URL` (default `http://localhost:8000`)
- **Validation**: Zod schemas for API response validation
- **Styling**: Tailwind CSS 4 with custom brand theme (CSS variables in `index.css`)
- **Font**: Everett-Medium custom font

### Path Aliases

- `@/*` → `src/*`
- `@assets/*` → `src/assets/*`

### Source Layout

- `src/api/` — API modules organized by domain (e.g., `monday/`). Each domain has `hooks.ts` (React Query), `schemas.ts` (Zod), `types.ts`
- `src/lib/api/` — Shared Axios instance and `ApiError` class with status helpers (`isUnauthorized()`, `isNotFound()`, etc.)
- `src/lib/query/` — QueryClient configuration
- `src/lib/validation/` — Zod error formatting utilities (`formatZodErrors`, `getFieldErrors`, `safeParseWithErrors`)
- `src/components/` — Shared components organized by feature (e.g., `dashboard/DashboardHeader.tsx`)
- `src/pages/` — Route-level pages. `DashboardLayout.tsx` wraps nested dashboard routes via `<Outlet />`
- `src/assets/index.ts` — Centralized asset registry for type-safe imports

### Routing Structure

```
/login              → LoginPage
/dashboard          → DashboardLayout (shared header + Outlet)
  /dashboard        → DashboardPage (index)
  /dashboard/home   → DashboardPage
  /dashboard/clientes → ClientesPage
/                   → redirects to /login
/*                  → redirects to /login
```

### Conventions

- Error messages in the API layer are in Spanish
- API modules follow the pattern: `types.ts` → `schemas.ts` → `hooks.ts` → `index.ts` (barrel export)
- Brand colors use `cafe-*` prefix in Tailwind (e.g., `cafe-green-dark`, `cafe-beige-light`)
- Barrel exports (`index.ts`) in `api/`, `components/`, `pages/`, `assets/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MCMike0399) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
