## aly-life-monolit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Start dev server with Turbopack
npm run build        # Production build
npm run lint         # ESLint (next lint)
npm start            # Start production server
```

No test framework is configured.

## Architecture

**aly-life** is a personal goal/task tracking app. The UI is in Spanish. Built with Next.js 16 (App Router), React 19, Supabase (auth + database), Tailwind CSS 3, and Framer Motion (`motion/react`).

### Auth Flow

- Google OAuth via Supabase. Login at `/login`, callback handled by `/auth/callback/route.ts`.
- Middleware (`src/middleware.ts`) enforces auth on all routes except `/login` and `/auth/*`. Redirects unauthenticated users to `/login` and authenticated users away from `/login`.
- Two Supabase clients: `src/lib/supabase/client.ts` (browser, `createBrowserClient`) and `src/lib/supabase/server.ts` (server components/route handlers, `createServerClient` via `@supabase/ssr`).

### Data Model

Two Supabase tables — `categories` and `tasks` (see `src/lib/types/database.ts`):
- **Category**: name, emoji, color, sort_order. Owns many tasks.
- **Task**: belongs to a category. Two types: `one_time` (checkbox, target=1) and `repetition` (counter with target_value). Status is derived client-side from `current_value` vs `target_value` and `due_date` (see `deriveStatus` in `src/lib/utils/progress.ts`). Stale statuses are reconciled on fetch.

### Client-Side Data Layer

All pages are `'use client'`. Data fetching and mutations happen through custom hooks that call Supabase directly:
- `useCategories` — fetches all categories + tasks, groups them, reconciles stale statuses.
- `useTasks(categoryId)` — CRUD + increment/decrement/toggle for a single category's tasks.
- `useAuth` — current user + sign out.
- `useTaskFilters(tasks)` — client-side search and status filtering.

### Pages

- `/` — Dashboard with bento grid of categories.
- `/category/[id]` — Category detail with task list, filters, and FAB for adding tasks.
- `/login` — Google OAuth login.

### UI Patterns

- Glassmorphism design: CSS classes `glass-card`, `glass-button`, `mesh-bg` defined in `globals.css`.
- Components use Radix UI primitives (Dialog, Select, Tooltip, Label) and `lucide-react` icons.
- Animations via `motion/react` (the `motion` package, not `framer-motion`).
- Color theming via CSS custom properties (HSL) with dark mode toggled by class. Managed by `ThemeProvider`.
- Path alias: `@/*` maps to `./src/*`.

### Environment Variables

Required (set in `.env.local`):
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DannerAly)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DannerAly)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
