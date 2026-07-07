## seaclouds-management-system

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

| Command | Purpose |
|---|---|
| `npm run dev` | Start Next.js dev server |
| `npm run build` | Production build |
| `npm run lint` | Run ESLint |
| `npm run update-types` | Regenerate Supabase TypeScript types into `utils/supabase/types.ts` |

No test framework is configured.

## Architecture

**Stack:** Next.js 16 (App Router, RSC), React 19, TypeScript, Supabase (auth + Postgres), TailwindCSS 3, shadcn/ui (new-york style), date-fns 4.

**What it does:** Employee timesheet management system. Employees log hours against sub-projects; admins manage projects, users, and view reports/stats.

### Data flow pattern

- **Pages** are async React Server Components that fetch data directly via Supabase queries, then pass props to client components.
- **Mutations** use Next.js Server Actions (`'use server'`) in `app/data/actions/` — no REST API routes. After mutations, `revalidatePath()` invalidates the cache and client components call `router.refresh()`.
- **No client-side data fetching** — all data loading happens server-side at render time.

### Auth & middleware

- Supabase email/password auth with three client variants:
  - `utils/supabase/client.ts` — browser client
  - `utils/supabase/server.ts` — server client (cookie-based SSR)
  - `utils/supabase/admin.ts` — admin client (service role key, for `inviteUser`)
- `proxy.ts` is the Next.js middleware (Next 16 convention) — redirects unauthenticated users to `/login`.
- `app/admin/layout.tsx` does a server-side role check, redirecting non-admins to `/`.

### Key directories

- `app/data/actions/` — all server actions, grouped by domain (`timesheet.ts`, `projects.ts`, `users.ts`, `stats.ts`), barrel-exported from `index.ts`
- `components/ui/` — shadcn/ui primitives (don't edit manually; use `npx shadcn@latest add`)
- `lib/utils.ts` — `cn()` helper (clsx + tailwind-merge)
- `docs/plans/` — project audit and implementation plans

### Database schema (Supabase)

Six tables: `profiles`, `projects`, `sub_projects`, `project_assignments`, `timesheet_entries`, `timesheet_submissions`. Two DB functions: `is_admin()` and `is_week_locked()`. One enum: `user_role` (`admin` | `employee`).

Types are auto-generated in `utils/supabase/types.ts` — run `npm run update-types` after schema changes. Row types are derived as `Database['public']['Tables']['<table>']['Row']`.

## Conventions

- Server actions: `'use server'` directive, placed in `app/data/actions/`, re-exported from `index.ts`.
- Client components: `'use client'` directive, co-located PascalCase files next to their page.
- Styling: Tailwind utilities + `cn()` for conditional classes. CSS variables for theming in `globals.css`.
- Path alias: `@/*` maps to project root.
- Supabase upsert calls sometimes use `as any` to work around type inference — this is intentional.

---
> Source: [tj0517/Seaclouds_management_system](https://github.com/tj0517/Seaclouds_management_system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
