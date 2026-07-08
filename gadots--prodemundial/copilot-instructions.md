## prodemundial

> A World Cup 2026 prediction pool app (USA · MEX · CAN). Users form groups ("prodes"), predict match results, accumulate points, and compete on a leaderboard. Mobile-first, UI is in Rioplatense Spanish (Argentina/Uruguay dialect).

# ProdeMundial — Claude Guide

## What is this project?

A World Cup 2026 prediction pool app (USA · MEX · CAN). Users form groups ("prodes"), predict match results, accumulate points, and compete on a leaderboard. Mobile-first, UI is in Rioplatense Spanish (Argentina/Uruguay dialect).

## Stack

- **Next.js 16** (App Router, `src/app`)
- **TypeScript** strict mode
- **Tailwind CSS** with dark palette (`#0a1628` base)
- **Supabase** (Auth + PostgreSQL + Realtime) — app works with **mock data** when Supabase is not configured
- **shadcn/ui** components in `src/components/ui/`

## Key folder structure

```
src/
  app/
    (app)/           ← protected routes with layout (nav, etc.)
      dashboard/     ← home: live matches, streak, token alerts
      predicciones/  ← match predictions + token selector
        especiales/  ← champion, top scorer, etc.
      tabla/         ← leaderboard with share button
      grupo/         ← prode info, invite code
      perfil/        ← user profile
    (auth)/          ← login / register
    admin/           ← admin panel (cookie-auth, not Supabase auth)
      (panel)/       ← protected admin routes
        dashboard/   ← stats overview
        users/       ← user management
        prodes/      ← prode management
        matches/     ← match list + manual sync button
      login/         ← admin login page
    api/
      admin/
        login/       ← sets admin session cookie
        logout/      ← clears cookie
        sync/        ← triggers match sync (accepts cookie OR Bearer CRON_SECRET)
        users/[id]/  ← DELETE user
        prodes/[id]/ ← DELETE prode
      cron/sync-matches/ ← legacy GET endpoint (still used by direct calls)
  components/
    nav.tsx          ← TopBar + BottomNav (5 items)
    flag.tsx         ← <Flag tla="ARG" /> renders country flag via flagcdn.com
    service-worker-register.tsx ← registers /sw.js for PWA
    ui/              ← shadcn components
  lib/
    types.ts         ← all TypeScript types (Phase, Match, Member, etc.)
    mock-data.ts     ← sample data (app works fully without Supabase)
    scoring.ts       ← points logic, tokens, streaks
    sync-matches.ts  ← core sync logic shared by cron and admin trigger
    admin-auth.ts    ← cookie-based admin auth (ADMIN_USERNAME/ADMIN_PASSWORD)
    supabase/
      queries.ts     ← client-side Supabase queries
      admin-queries.ts ← server-side admin queries (service role)
public/
  sw.js              ← static service worker (PWA, no build integration needed)
  manifest.json      ← Web App Manifest
  icons/             ← PWA icons (192, 512, maskable)
supabase/
  migrations/        ← run in order: 001 → 007
```

## Scoring mechanics

### Base points by phase

| Phase | Exact | Correct winner | Draw |
|-------|-------|----------------|------|
| Group stage | 3 pts | 1 pt | 2 pts |
| Round of 32 | 6 pts | 2 pts | — |
| Round of 16 | 10 pts | 4 pts | — |
| Quarter-finals | 18 pts | 6 pts | — |
| Semi-finals | 30 pts | 10 pts | — |
| Third place | 30 pts | 10 pts | — |
| Final | 50 pts | 20 pts | — |

### Multiplier tokens (⚡2x / 🔥3x / 💥5x)

- Each user gets **1 token of each type** when joining a prode
- Can be applied to **any match** (no phase restriction)
- If unused before end of group stage → **expire** (decay)
- Each token can only be used **once**
- Managed in `mock-data.ts` (`MOCK_MY_TOKENS`) and the predictions page
- In the UI, tokens are called "Potenciadores" — TypeScript names (`MultiplierToken`, `TokenMultiplier`) are unchanged

### Hot streak

- +3 pts automatic bonus on next correct prediction with **3 consecutive correct** predictions
- +8 pts with **5+** consecutive
- Streak resets on any wrong prediction or no-score prediction
- See `streakBonusPoints()` in `scoring.ts`

### Special predictions

- Champion: 60 pts · Top scorer: 40 pts · Finalist: 35 pts · Third place: 25 pts · Most goals country: 20 pts
- Locked once the tournament starts
- Page: `/predicciones/especiales`

## Code conventions

- **UI always in Rioplatense Spanish** (vos, pts, guardado, etc.)
- Components: arrow functions + explicit `"use client"` where needed
- Do not use `jokerUsed` (replaced by `multiplier: TokenMultiplier`)
- Token palette: blue=2x, orange=3x, purple=5x
- Mock data in `mock-data.ts` is used as fallback; in production everything comes from Supabase
- Avoid circular imports: `types.ts` imports nothing from the project
- Flags: always use `<Flag tla="..." />` component (flagcdn.com images) — never emoji flags (broken on Windows)

## Commands

```bash
npm run dev      # development at localhost:3000 (Turbopack)
npm run build    # production build (must pass with zero TS errors)
npm run lint     # ESLint
npm run test     # Vitest unit tests
npm run test:e2e # Playwright E2E tests
```

## Environment variables (production)

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
FOOTBALL_DATA_API_KEY=      # football-data.org (free tier)
CRON_SECRET=                # shared secret: GitHub Actions ↔ /api/admin/sync
ADMIN_USERNAME=             # admin panel username (default: "admin")
ADMIN_PASSWORD=             # admin panel password — REQUIRED to enable admin
```

Without these variables the app runs with mock data.

## Database

Migrations live in `supabase/migrations/` and must be run in order (001–007):

- `001_initial_schema.sql` — full schema, RLS, triggers, `calculate_match_points()`
- `002_idempotent_points_and_decay.sql` — idempotent scoring + token decay
- `003_fix_rls_recursion.sql` — fixes infinite recursion in RLS policies
- `004_fix_prode_admin_select.sql` — fixes admin SELECT policy on prodes
- `005_add_delete_policies.sql` — RLS policies for DELETE operations
- `006_fix_leaderboard_security_invoker.sql` — leaderboard view uses `security_invoker=true` (fixes Supabase security alert)
- `007_add_third_place_phase.sql` — adds THIRD_PLACE to phase CHECK constraints and scoring function

Key DB objects:
- Trigger `on_auth_user_created` → auto-creates profile
- Trigger `on_prode_member_added` → auto-assigns 3 tokens
- Function `calculate_match_points(match_id)` → calculates points with multiplier + streak bonus
- View `public.leaderboard` → cached rankings (security_invoker = true)

## Admin panel

- Route: `/admin` — uses cookie-based auth, **not** Supabase auth
- Protected by `ADMIN_USERNAME` / `ADMIN_PASSWORD` env vars
- `/api/admin/sync` accepts either admin cookie (manual button) OR `Authorization: Bearer CRON_SECRET` (GitHub Actions)
- If `ADMIN_PASSWORD` is unset, all admin routes return 401

## Match sync

- GitHub Actions (`.github/workflows/sync-matches.yml`) calls `/api/admin/sync` via Bearer token
- Runs every 5 minutes during the World Cup (Jun 11–Jul 19 2026), daily otherwise
- Uses `football-data.org` competition ID 2000 (FIFA World Cup)
- `mapStage()` in `sync-matches.ts` maps API stage names to our Phase type

## PWA

- `public/sw.js` — static service worker, no build integration, works with Turbopack
- `public/manifest.json` — Web App Manifest with display standalone
- `ServiceWorkerRegister` client component registers the SW on first load
- Icons generated via `scripts/generate-icons.mjs` (requires `sharp`)

## Current project state

- Full UI with mock data — works without Supabase
- Supabase auth + DB fully wired (queries.ts / admin-queries.ts)
- football-data.org sync operational
- Admin panel operational
- PWA installable on Android/iOS
- **Color palette**: dark navy `#0a1628` base, amber accent (`amber-500`) throughout — replaced all green accents
- **Leaderboard (`/tabla`)** has two views: total points table + 📊 Stats tab (best streak, best phase, token status, points today per member)
- **Reveal post-partido**: in dashboard MatchCards, a "Ver predicciones del prode" button expands inline for LIVE and FINISHED matches — shows all members' predictions and points. Query: `getMatchPredictions(matchId, prodeId)` in `queries.ts`; mock data in `MOCK_MATCH_PREDICTIONS` in `mock-data.ts`
- **Perfil (`/perfil`)**: edit display name (calls `updateDisplayName` + `refreshUser` from context) + change password inline. Avatar/initials button in TopBar (`showProfile` prop) links to `/perfil` from all pages
- **Wildcards (weekly challenges) removed from MVP.** The `/desafios/` folder was deleted and its nav slot replaced by Especiales. `WildcardChallenge` type and related data remain in `types.ts` and `mock-data.ts` in case it's reactivated.
- No negative scoring (by design, won't be implemented)
- Production URL: `https://elprode.vercel.app`

---
> Source: [gadots/ProdeMundial](https://github.com/gadots/ProdeMundial) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
