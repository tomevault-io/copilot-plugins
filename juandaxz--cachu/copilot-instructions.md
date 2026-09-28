## cachu

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start dev server at localhost:3000
npm run build    # Production build
npm run lint     # Run ESLint
```

No test runner is configured. `next.config.mjs` sets `typescript.ignoreBuildErrors: true` and `eslint.ignoreDuringBuilds: true`, so build errors from TS/lint are suppressed — but lint issues still surface with `npm run lint`.

## Setup

Create `.env.local` with:
```
NEXT_PUBLIC_SUPABASE_URL=...
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
```

Run the schema in the Supabase SQL Editor using `supabase/schema.sql`. Note: that file only covers habits/anti-habits/todos — the `profiles` (with `ical_url`) and `water_logs` tables used by the calendar-import and Focus features (see below) are not defined there; check the live Supabase project's schema before assuming `schema.sql` is authoritative.

## Architecture

**Cachuflin** (branded "Balancepol" in the UI) is a personal productivity app — habit tracking, anti-habit/addiction tracking, todos, a Pomodoro/water-tracking focus page, and university calendar (.ics) integration — built with Next.js 15 App Router + Supabase.

### Stack

- Next.js 15 App Router, React 19, TypeScript 5
- Supabase (PostgreSQL + Auth + Row-Level Security)
- Google OAuth via Supabase Auth
- Tailwind CSS + Radix UI primitives + Lucide icons
- date-fns for date math

### Route Structure

```
src/app/
├── (app)/               # Protected routes — grouped layout with sidebar/mobile nav
│   ├── page.tsx         # Dashboard
│   ├── habits/          # Habits + anti-habits, unified via ?tab=positivos|dejar
│   ├── todos/           # Task management (kanban + calendar view via ?view=)
│   ├── calendar/        # Standalone calendar page
│   ├── focus/           # Pomodoro timer + daily water tracker
│   ├── stats/           # Heatmap statistics
│   └── settings/        # iCal URL configuration
├── auth/callback/       # OAuth code exchange
├── login/               # Google OAuth entry point
└── actions/             # Server Actions (habits.ts, anti-habits.ts, todos.ts, calendar.ts, water.ts)
```

The sidebar/bottom nav (`src/components/layout/app-layout.tsx`) only links to Inicio, Hábitos, Tareas, Focus, and Stats — anti-habits live inside `/habits` as a tab, and the standalone `/calendar` route is not in the nav (calendar data surfaces on the dashboard and inside `/todos` instead).

### Data Flow

Pages are **async Server Components** that fetch directly from Supabase. Mutations use **Server Actions** (`'use server'`) with `revalidatePath()` for cache invalidation. There is no client-side global state or data-fetching library — keep new features in this pattern.

### Auth

`src/middleware.ts` protects all routes, redirecting unauthenticated users to `/login`. The Supabase client has two variants: `src/lib/supabase/client.ts` (browser) and `src/lib/supabase/server.ts` (server/cookies). Every table uses RLS with a single `auth.uid() = user_id` policy.

### Domain Models

Core entities, each with a checkins/related table (see `supabase/schema.sql`):

| Entity | Table | Related Tables |
|--------|-------|----------------|
| Habits | `habits` | `habit_checkins` (daily value/count) |
| Anti-Habits | `anti_habits` | `anti_habit_checkins`, `anti_habit_journal` |
| Todos | `todos` | `todo_categories` |

Two more tables are used by the app but **not** in `schema.sql` (create manually / verify against the live project): `profiles` (`ical_url` column, feeds `src/lib/ical.ts` for calendar import) and `water_logs` (`user_id, date, count`, used by `src/app/actions/water.ts`).

Habit types: `'boolean'` (checked/unchecked) or `'count'` (numeric target). Todo urgency: `'low' | 'medium' | 'high' | 'risk'`. Todo status: `'pending' | 'in_progress' | 'done'`.

All TypeScript interfaces are in `src/lib/types.ts`.

### Key Utilities (`src/lib/utils.ts`)

- `today()` — current date as `'yyyy-MM-dd'`, computed in the `America/Guayaquil` timezone (not the server's local timezone)
- `calcHabitStreak()` / `calcAntiHabitStreak()` — streak logic
- `buildYearGrid()` / `buildHistoryGrid()` — heatmap data structures (52-week and rolling-window grids)
- `DEFAULT_CATEGORIES` — 4 seeded todo categories

### Calendar import

`src/lib/ical.ts` is a small hand-rolled `.ics` parser (`fetchCalendarEvents`, `upcomingEvents`) — the `ical.js` npm dependency in `package.json` is **not** actually used anywhere in `src/`. Don't assume `ical.js` APIs apply here.

### UI Conventions

- Path alias: `@/*` maps to `src/*`
- Shadcn-style UI primitives live in `src/components/ui/`
- CSS variables drive the color/radius theme (see `tailwind.config.ts`)
- UI text is in Spanish; code/variables are in English

---
> Source: [juandaxz/Cachu](https://github.com/juandaxz/Cachu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
