## cc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — start the dev server (http://localhost:3000)
- `npm run build` — `prisma generate` + `next build`
- `npm run lint` — Next.js ESLint
- `npm run check` — typecheck (`tsc --noEmit`)
- `npm run prisma:migrate` — create/apply a dev migration; `npm run prisma:deploy` — apply in prod
- `npx prisma db seed` — seed the default admin from `.env` (`DEFAULT_ADMIN_EMAIL` / `DEFAULT_ADMIN_PASSWORD`)

There is no unit-test framework. The closest thing is `node scripts/test-game-flow.mjs`, an assertion-based simulation of full game flow (regulation → redemption → overtime) that runs against the real database configured in `.env` (it tags its games with location `TEST_SIM_FLOW`).

Postgres is required (`DATABASE_URL` in `.env`). Copy `.env.example` to get started.

## What this app is

Century Cup league tracker: a mobile-first Next.js 14 App Router app (TypeScript, Tailwind, Prisma/Postgres, NextAuth credentials, SWR) for a 100-cup pong league — auth, Excel roster/schedule import, a live game console for stat-taking, and postgame/season stats.

Layout: `src/app/(auth)` sign-in/sign-up, `src/app/(app)` authenticated pages, `src/app/api` route handlers, `src/lib` domain logic, `src/components` client components. Path alias `@/*` → `src/*`. The `.xlsx` files in the repo root are import data, not junk.

## Core architecture: event-sourced games

`docs/game-definitions.md` is the canonical spec for game-state semantics — read it before touching game logic, and keep it updated.

- For `TRACKED` games, the `ShotEvent` log is the source of truth. `GameState` (cups remaining, possession, shooter index, phase, status) is a derived cache. `recomputeGameState()` in `src/lib/game-state.ts` rebuilds it from the event log inside a transaction — undo and admin corrections work by editing events and then recomputing, never by mutating `GameState` directly. Admin corrections to finished games pass `preserveFinalStatus` so the game stays `FINAL`.
- Cup arithmetic: both teams start at 100, clamped 0–100. Makes by the offense reduce the *opponent's* cups. Make types are `TOP_REGULAR`, `TOP_ISO`, `BOTTOM_REGULAR`, `BOTTOM_ISO`; `PULL_HOME`/`PULL_AWAY` adjust a specific side's rack by `cupsDelta` (negative delta adds cups) and are not shots.
- Turn flow: a `Turn` row per rack with `shootersJson` (the 1–6 shooting order). Turns are 6 shots; 2+ makes earns a bonus turn for the same offense, otherwise possession flips. Phases: `REGULATION` → `REDEMPTION` (defense shoots until a miss once a team hits 0; the redemption shooter index advances on misses, not shots) → `OVERTIME`. An overtime game tied 0–0 stores the winner in `GameState.possessionTeamId`.
- Live console (`src/components/live-console.tsx`) drives everything through `/api/games/[id]/events`, `/undo`, `/advance`, `/finalize`, `/state`.

## LEGACY vs TRACKED stats

`LEGACY` games are historical imports whose source of truth is aggregate rows (`LegacyPlayerStat`/`LegacyTeamStat`), and their `homeCupsRemaining`/`awayCupsRemaining` fields have *inverted* winner semantics versus tracked games. Never compare remaining cups directly — always use `winnerFromRemaining`/`winnerFromGameState` from `src/lib/stats.ts`, which handle both sources. Imports must never overwrite a tracked game that occupies the same schedule slot.

## Other conventions

- **Stats formulas** live only in `src/lib/stats.ts`: `boxScore`, `baseRatingStats` (base weights), `advancedStats` (tempo rating with temporal scaling on cups remaining), `defaultMultipliers`. Pages should compute ratings through these, not reimplement weights.
- **Auth**: NextAuth credentials with JWT sessions; `role` (`ADMIN`/`USER`) is attached in the callbacks in `src/lib/auth.ts` (session type augmented in `src/types/next-auth.d.ts`). Only a game's `statTaker` may post events to it; admin API routes check `role === 'ADMIN'` and log to `AdminAuditLog` via `src/lib/admin-audit.ts`.
- **Seasons** are named `F<year>`/`S<year>` (e.g. `S2026`). `src/lib/season.ts` owns sorting and resolving the selected season from the `?season=` URL param (defaults to the current term, `all` supported).
- **Player identity across seasons**: players are matched by email and by `PlayerAlias` (nickname) rows, built from `Name_email_mapping.xlsx`. Historical import: `node scripts/import-historical.mjs` (optionally `--file "SOMIL S2026.xlsx"`); current-season import goes through the admin UI at `/admin/import`.
- **Weekly recap** (`src/lib/ai.ts`) uses `GEMINI_API_KEY` and falls back to a templated recap when unset.

---
> Source: [svinod26/CC](https://github.com/svinod26/CC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
