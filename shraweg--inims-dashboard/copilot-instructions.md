## inims-dashboard

> pnpm dev          # starts dev server on port 6061

# INIMS Dashboard — Agent Guide

## Quick start

```bash
pnpm dev          # starts dev server on port 6061
pnpm build        # production build
pnpm start        # prod server on port 6061
```

No test, lint, or typecheck scripts exist. Run `npx tsc --noEmit` to typecheck manually.

## Stack

- **Next.js 16** App Router, TypeScript strict, React Compiler enabled (`reactCompiler: true` in next.config.ts)
- **Tailwind CSS v4** via `@tailwindcss/postcss`, **shadcn/ui** (radix-nova style), Tailwind v4 `@theme` directives in globals.css
- **TanStack Query v5** for all server state. Default staleTime 5 min, retry disabled. Devtools enabled.
- **React Hook Form + Zod** for all forms (Zod v4)
- **recharts** for charts
- **react-i18next** for i18n, locales at `src/i18n/locales/{en,ne}/`
- **axios** client at `src/lib/api/client.ts` — auto-attaches `access_token` cookie as Bearer token, handles 401 → refresh flow
- **lucide-react** icons, **Material Symbols** from Google Fonts CDN (loaded in root layout)

## Architecture

### Path alias
`@/*` maps to `./src/*` (`@/components/...`, `@/lib/...`, etc.)

### Route groups
- `(auth)/` — login page only
- `(dashboard)/` — all authenticated pages, wrapped in sidebar+topbar layout
- Root `page.tsx` redirects to `/dashboard`

### Auth flow
- Login via Next.js API route at `src/app/api/auth/login/route.ts` — sets `access_token` + `refresh_token` httpOnly cookies
- Middleware (`src/middleware.ts`) checks cookies on every request; allows refresh-token-only users through to avoid jarring redirects
- `useAuth` hook fetches `/users/me`; stores optimistic user in `localStorage` key `inims-user-presumed`
- Axios interceptor auto-refreshes tokens on 401, redirects to `/login` on failure

### RBAC
- `PermissionGuard` component wraps sensitive UI; passes `permission` or `permissions` prop
- `usePermissions()` hook — checks `hasPermission`, `hasAnyPermission`; SUPER_ADMIN/ADMIN bypass checks
- Permission convention: `resource:action` (e.g. `users:view`, `roles:create`)

### Directory conventions
- `src/app/(dashboard)/[feature]/` — thin page entry points (list, create, `[id]/`, `[id]/update`)
- `src/features/[feature]/` — actual components: tables, forms, hooks
- `src/components/common/` — shared: `data-table`, `PermissionGuard`, `page-header`, `searchable-select`, `multi-select`, `confirm-dialog`, form fields, status badges
- `src/components/layout/` — sidebar (dynamic import, `ssr: false`), topbar, user menu, locale switcher
- `src/lib/api/` — axios client, `ENDPOINTS` constant object, feature-specific API modules
- `src/lib/validations/` — Zod schemas
- `src/types/` — TypeScript types/interfaces
- `src/hooks/` — `use-auth`, `use-permissions`, `use-locale`, `use-master-data`
- `src/providers/` — i18n (client-only, skips SSR), QueryClientProvider, AuthProvider (thin wrapper)
- `src/config/` — navigation config (JSON + TS with lucide icon mapping)

### API endpoints
All defined as constants in `src/lib/api/endpoints.ts`. Backend runs at `http://localhost:6060/api/v1` (from `.env`).

### Design tokens
- Primary: Crimson Red `oklch(0.536 0.232 15.65)`, Secondary: Deep Blue `#004b8e`
- All colors/spacing/radius via CSS variables in `src/app/globals.css` — never hardcode values
- Custom sidebars use `--color-sidebar-*` tokens, not the standard shadcn sidebar

## Coding rules (always)

- No `any` types
- No `console.log`
- No raw `fetch` in components — use `apiClient` from `@/lib/api/client`
- No `useEffect` + `useState` for data fetching — use TanStack Query
- All forms: react-hook-form + Zod
- All tables: shadcn DataTable (`src/components/common/data-table/`)
- UI components from shadcn/ui, customized only via `className`
- Components call API through custom hooks, never directly
- Sensitive UI wrapped in `PermissionGuard`
- No modal CRUD — dedicated pages for Create/Update/View
- Page components kept thin; logic in `src/features/`

## Existing instruction files (read these for deeper context)

- `.agent/SKILL.md` — full coding conventions, design system, module checklist references
- `.agent/references/api-client.md` — API client setup, interceptors, error handling
- `.agent/references/auth-flow.md` — login, token refresh, logout, cookie management
- `.agent/references/charts.md` — recharts usage, responsiveness, theming
- `.agent/references/conventions.md` — naming, imports, file organization, exports
- `.agent/references/dependencies.md` — all third-party libs with versions and purpose
- `.agent/references/design-system.md` — colors, typography, spacing, component styling
- `.agent/references/error-handling.md` — error boundaries, query error handling, toasts
- `.agent/references/folder-structure.md` — complete directory tree with descriptions
- `.agent/references/forms.md` — react-hook-form + Zod patterns, form field components
- `.agent/references/i18n.md` — locale setup, translation files, usage patterns
- `.agent/references/masters.md` — master entity CRUD patterns
- `.agent/references/new-module-checklist.md` — step-by-step guide for adding a feature
- `.agent/references/new-project-setup.md` — initial project scaffolding
- `.agent/references/rbac.md` — permission system, PermissionGuard, role checks
- `.agent/references/tables.md` — DataTable component, column defs, pagination, search
- `.agents/SKILL.md` — CRUD routing patterns, RBAC enforcement, visual standards

---
> Source: [SHRAWEG/inims-dashboard](https://github.com/SHRAWEG/inims-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
