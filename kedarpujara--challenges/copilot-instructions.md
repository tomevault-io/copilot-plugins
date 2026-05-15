## challenges

> Social accountability + habit-tracking web app. Users create or join daily challenges with custom metrics, check in each day, track streaks, and compare with friends or public participants. Includes bot participants to keep small challenges alive.

# Challenges — CLAUDE.md

Social accountability + habit-tracking web app. Users create or join daily challenges with custom metrics, check in each day, track streaks, and compare with friends or public participants. Includes bot participants to keep small challenges alive.

_Prefix: `CH` · Repo: `~/projects/Challenges` · Status: **inactive** (last commit 2026-01-24)._

## Stack

| Layer | Choice |
|-------|--------|
| Framework | Next.js 14.2 (App Router) |
| Language | TypeScript 5 |
| Styling | Tailwind CSS 3.4 — dark stone palette, orange accent |
| Icons | Lucide React |
| Charts | Recharts 3 |
| Data | Supabase (Postgres + Auth + Storage) |
| Server state | TanStack React Query 5 |
| Client state | Zustand 5 (offline queue + active challenge) |
| Forms | React Hook Form 7 + Zod 4 |
| PWA | `next-pwa` |
| Deploy target | Vercel (not confirmed live currently) |

## Repo layout

```
src/
  app/
    (auth)/
      signup/, login/, callback/       Supabase OAuth flow
    (main)/
      dashboard/                       Overview
      challenges/                      Browse + join + create
      checkin/                         Daily entry form
      profile/                         User profile
      settings/
  components/
    ui/                                Button, Input, Card, Modal, Tabs, Progress, Avatar, PhotoUpload
    challenge/                         ChallengeCard
    navigation/                        BottomNav (mobile-first)
  hooks/
    useDailyEntry.ts                   React Query + Supabase
    useChallenge.ts
  stores/
    appStore.ts                        Zustand — offline queue, profile, active challenge id
  lib/
    supabase/                          client.ts, server.ts, middleware.ts
    providers.tsx                      QueryClient provider
  middleware.ts                        Supabase session refresh
supabase/
  migrations/
    00001_initial_schema.sql
    00002_fix_invite_code_rls.sql
    00003_storage_bucket.sql
```

## Run / dev / build

```bash
npm install
npm run dev          # :3000
npm run build
npm start
```

## Environment

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=            # server-only, optional for elevated ops
```

## Data model (Supabase — RLS on every table)

- **`profiles`** — 1:1 with `auth.users`. `username`, `display_name`, `avatar_url`, `timezone`, streaks, challenge counts. Auto-created via trigger on signup.
- **`challenge_templates`** — reusable blueprints. `metrics` stored as JSONB.
- **`challenges`** — active instances. `owner_id`, `start_date`, `end_date`, `invite_code`, visibility (`public` / `private`), `metrics` JSONB.
- **`challenge_participants`** — join table. Supports bot participants: `is_bot`, `bot_type` (`consistent` | `human` | `struggling`). Tracks streak.
- **`daily_entries`** — per-participant per-day. `metrics_data` JSONB, `is_complete`, pass/fail counts, optional `photo_url` (Supabase Storage).
- **`user_badges`** — earned badges linked to challenges.

Triggers handle auto-profile creation and automatic streak recalculation.

## Deploy

- **Target:** Vercel (Next.js native). Not actively deployed — treat as cold.
- **PWA:** `next-pwa` configured. Manifest + service worker built for installable mobile.

## Key patterns

- **Offline-first:** Zustand `appStore` keeps a queue of failed/offline entries and flushes when online.
- **Server-side session:** Middleware refreshes the Supabase token; most pages call `getSession()` (cookie-cached) rather than `getUser()` for speed.
- **React Query keys** are granular (`['daily-entry', id]`, `['challenge', id]`, `['challenges']`). Every mutation invalidates the matching key set.
- **Forms** use React Hook Form + Zod schemas — validation lives alongside the form, not scattered through components.
- **RLS does the access control** — server/client share the same query shape; row-level policies enforce "only your rows."
- **Stone palette + orange accent** — Tailwind `stone-*` for backgrounds, `orange-500` for primary action.

## DO NOT casually change

- **RLS policies** — loosening exposes other users' entries.
- **`auth.users` → `profiles` trigger** — removing it breaks signup silently.
- **Zustand offline queue schema** — users may have queued entries on disk; breaking format loses them.
- **Migration files** — never edit existing migrations; add new numbered ones.
- **Bot participant logic** — three `bot_type` flavors have different cadences; refactoring means re-tuning.

## Known gotchas

- **Invite-code RLS fix (migration 00002):** the initial policy blocked reading an invite code before joining. Don't revert.
- **Timezone handling:** `profiles.timezone` matters — check-in "today" is computed in user local time. Server math must respect it.
- **Supabase Storage bucket** created in migration 00003 — photo uploads depend on it existing in the target project.
- **React Query cache hydration** may mismatch if session middleware fails — always test after auth changes.

## Status / revival notes

- Last commit: `1b24330 Preset metrics` (2026-01-24).
- Schema is finalized (3 migrations). No known incomplete features in code.
- Remote is HTTPS (`https://github.com/kedarpujara/Challenges`), not SSH — if automated pushes are needed, either switch to SSH or configure a token.
- To revive: fresh Supabase project, run all 3 migrations in order, fill env, `npm run dev`.

---
> Source: [kedarpujara/Challenges](https://github.com/kedarpujara/Challenges) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
