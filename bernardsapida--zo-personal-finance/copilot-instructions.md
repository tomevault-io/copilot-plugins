## zo-personal-finance

> - **Dev:** `npm run dev` (port 3000)

# TanStack Start Monolith Template

## Commands

- **Dev:** `npm run dev` (port 3000)
- **Build:** `npm run build`
- **Lint/Format/Typecheck:** `npm run check` (Biome — runs all)
- **DB migrate:** `npm run db:migrate`
- **DB push (no migration):** `npm run db:push`
- **DB generate client:** `npm run db:generate`
- **DB studio:** `npm run db:studio`

## Stack

- **Framework:** TanStack Start (React 19, file-based routing via TanStack Router)
- **UI:** HeroUI v3 + Tailwind CSS v4 — compound component pattern, no Provider needed
- **Icons:** Lucide React
- **Forms:** React Hook Form + Zod v4 + `@hookform/resolvers`
- **Server state:** TanStack Query
- **Client state:** Zustand
- **API:** tRPC v11 (routers in `src/integrations/trpc/`)
- **Auth:** Better Auth (server in `src/features/auth/utils/better-auth.ts`, client in `auth-client.ts`)
- **ORM:** Prisma v7 + PostgreSQL (`src/lib/prisma.ts`)
- **Env validation:** T3Env (`src/env.ts`) — import `env` from there, never `process.env` directly
- **Linter/Formatter:** Biome

## Environment Variables

All vars go in `.env.local` (loaded via `dotenv-cli`):

```
BETTER_AUTH_SECRET=
BASE_URL=http://localhost:3000
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
VITE_APP_TITLE=        # optional
SERVER_URL=            # optional
```

## Project Structure

```
src/
  components/
    ui/          # Display-only, no RHF, each has className prop
    form/        # RHF-bound, generic <T extends FieldValues>, useController
    composite/   # Composed of ui + form (e.g. DataTableCard, FormModal, AppForm)
    feedback/    # Toast, QueryError
  features/      # Feature-based: auth/, user/, admin/ — each has api/, hooks/, validations/
  integrations/
    trpc/        # init.ts, router.ts, react.ts, routers/
  hooks/         # Shared hooks (use-debounce, useRouteBreadcrumbs, etc.)
  lib/           # prisma.ts, schemas/, status-maps/
  store/         # Zustand stores (auth.store.ts, ui.store.ts)
  config/        # navigation.config.ts, seo.config.ts, query-client.ts
  types/         # Shared TS types
  utils/         # cn.ts (class merging), format.ts, config.ts
  routes/        # TanStack Router file-based routes
  env.ts         # T3Env validated environment
```

## Component Conventions

**ui/** — Presentational only, no data fetching, no RHF:
- Always accept `className?: string`
- Derive prop types from HeroUI via `ComponentProps<typeof X>["prop"]`
- Every folder has an `index.ts` barrel export

**form/** — Always RHF-bound:
- Generic: `<T extends FieldValues>` with `name: Path<T>`, `control: Control<T>`
- Use `useController({ name, control })` — never `register`
- Call `field.onBlur()` alongside `field.onChange()` where applicable
- Hardcoded `className="w-full"` form fields accept `className` prop with `?? "w-full"` default

**composite/** — Composed components (DataTableCard, FormModal, AppForm)

**Breadcrumbs:** Use `useRouteBreadcrumbs` hook + add `staticData: { breadcrumb: "Label" }` to route definitions

**Status chips:** Use `StatusChip` with a `statusMap` from `src/lib/status-maps/`

## HeroUI v3 Rules

- **Compound components always:** `<Card><Card.Header>` not `<Card title="x">`
- **No Provider** — HeroUI v3 needs none
- **Semantic variants:** `primary`, `secondary`, `tertiary`, `danger`, `ghost` — no raw colors
- **Use `onPress`** not `onClick`
- **Always fetch docs before implementing** a new HeroUI component:
  ```bash
  node .claude/skills/heroui-react/scripts/get_component_docs.mjs ComponentName
  ```

## Code Rules (Biome enforced)

- JSX attributes must be **alphabetically sorted**
- Use `Number.isNaN()` — never `isNaN()`
- No `as any` — use `Record<string, unknown>` or proper generics
- No unused variables or parameters
- `forEach` callbacks must not return values — use `for...of` if you need early returns

## Path Aliases

`@/*` resolves to `src/*`:
- `@/components/ui` → `src/components/ui`
- `@/integrations/trpc/router` → `src/integrations/trpc/router`

## tRPC Pattern

- Routers live in `src/integrations/trpc/routers/` and `src/features/*/api/`
- Use `useTRPC` from `src/integrations/trpc/react.ts` in components
- Procedures use TanStack Query integration (`useSuspenseQuery`, `useMutation`)

## Auth Pattern

- Session check: `src/features/auth/lib/session.ts`
- Client-side: `authClient` from `src/features/auth/utils/auth-client.ts`
- Auth store: `src/store/auth.store.ts` (Zustand)
- Protected routes check session in loader, redirect to `/sign-in` if unauthenticated

## Skills

- **HeroUI v3:** `.claude/skills/heroui-react/` — use scripts to fetch component docs before implementing

---

1) Audit logs should have delete actions for each record (include alert dialog for confirmation)
2) Finance add record, TextField when required should have isRequired props. The notes should have the same style when hovered, etc... same with TextField.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BernardSapida)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BernardSapida)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
