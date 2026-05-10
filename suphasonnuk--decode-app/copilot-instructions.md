## decode-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview
Personal daily tracking app ("DECODE") with a TypeScript/Express backend and React/Vite frontend. Tracks daily goals across three pillars: Work, Future, Body. Includes nutrition tracking, coffee logging, AI coaching (Claude API), push notifications, presence/friends system, achievements, and weekly reviews.

## Architecture
- **Backend**: Express server (`src/`) with BigQuery as the database. Routes split by domain in `src/routes/` (core, nutrition, coffee, coach, challenge, push, presence, decoded)
- **Frontend**: React 18 + Vite (`client/`) with TanStack Query, Recharts, Framer Motion
- **Deployment**: Docker multi-stage build → Google Cloud Run (asia-southeast3). Production uses `npx tsx` (not compiled JS)
- **Auth**: Shared secret via `APP_SECRET` env var, timing-safe SHA-256 comparison in `route-utils.ts`
- **User ID**: Device-scoped UUID generated client-side, stored in localStorage, passed via `x-user-id` header

## Key Commands
```bash
# Development (run these in separate terminals)
npm run dev:server          # Express with nodemon + ts-node on :3000
npm run dev:client          # Vite dev server (proxies /api → :3000)

# Build
npm run build               # Build client + server
npm run build:client        # Vite build → dist/public
npm run build:server        # tsc → dist/server

# Test
npm test                    # Vitest (route-utils, nutrition reconciliation)
npm run test:watch          # Vitest in watch mode

# Production
npm start                   # node dist/server/index.js
```

## BigQuery Data Patterns
BigQuery is append-only. This shapes all data operations:
- **Latest version**: All reads use `ROW_NUMBER() OVER (PARTITION BY [key] ORDER BY submitted_at DESC) WHERE rn = 1` to get the most recent row
- **Edits**: Insert a new row with the same `entry_id` + `edited_at` timestamp. The partition query picks the latest
- **Deletes**: Insert a tombstone row with `deleted = TRUE`. Queries filter `WHERE deleted IS NULL OR deleted = FALSE`
- **Tables**: `daily_log`, `user_profile`, `nutrition_log`, `coffee_log`, `push_subscriptions`, `user_presence`, `ai_cache`
- All queries use parameterized params (never string interpolation for user input)

## AI Endpoints Architecture
Three endpoints call the Anthropic API, each with different patterns:
- **`/api/coach`** — Single Sonnet call. Feeds 30 days of BigQuery log data + user profile. Rate limited 5 req/min
- **`/api/nutrition/analyze`** — 3-pass pipeline: (1) Sonnet estimator with chain-of-thought, (2) Sonnet verifier as independent critic, (3) Haiku for health impact text. If estimates diverge >15%, averages them. Rate limited 20 req/min
- **`/api/challenge`** — Single Sonnet call with behavioral science framing. Client caches response per day
- **`/api/decoded/portrait`**, **`/api/weekly-story`** — Additional Sonnet calls for analytics

All AI calls use raw `fetch()` to `api.anthropic.com` with 30-35s abort timeouts. Responses are cached server-side in BigQuery `ai_cache` table keyed by `(user_id, cache_type, cache_date)` — survives localStorage clears and works across devices.

## Frontend Patterns
- **Lazy loading**: All tabs except Auth, Onboarding, DailyStatus, and DecodeLogo are `React.lazy()` with Suspense
- **Tab navigation**: Primary tabs (Home, Today, Night, Decode) in bottom bar; secondary tabs (Food, Coffee, Week, Trends, Coach, Anchors, Help) in "More" menu
- **State**: No global state manager. localStorage for persistence, sessionStorage for per-session flags, TanStack Query for server state
- **Midnight refresh**: Timer remounts daily components at 00:00:05 local time via key increment
- **Presence heartbeat**: Sends heartbeat every 60s while app is open
- **Swipe navigation**: Touch gestures navigate between primary tabs
- **Offline support**: Service worker with IndexedDB background sync for failed log submissions. Offline banner + pending sync count shown via `useOfflineStatus` hook in App.tsx

## Conventions
- TypeScript with relaxed tsconfig (`strict: false`, `noImplicitAny: false`)
- Server uses CommonJS modules; client uses ESM
- `safeStr()` sanitizes all user input before BigQuery insertion (strips backticks, quotes, semicolons)
- Date handling: local date strings (`YYYY-MM-DD`), Bangkok UTC+7 aware
- API routes all under `/api/*`, health endpoint is public, rest require auth
- In-memory rate limiter in `route-utils.ts` (not distributed — single instance only)
- Zod validation in `client/src/lib/schemas.ts`

## Environment Variables
- `GCP_PROJECT_ID` — required, BigQuery project
- `BQ_DATASET` — default: `decode_tracker`
- `BQ_TABLE` — default: `daily_log`
- `GCP_KEY_FILE` — local dev only, path to service account key
- `APP_SECRET` — shared auth token (omit for open dev mode)
- `ALLOWED_ORIGIN` — CORS restriction (omit for open dev)
- `ANTHROPIC_API_KEY` — for AI coaching/nutrition/challenge endpoints
- `VITE_VAPID_PUBLIC_KEY` — client-side push notification key
- `VAPID_PRIVATE_KEY`, `VAPID_PUBLIC_KEY`, `VAPID_EMAIL` — server-side push config
- `SCHEDULER_SECRET` — authenticates Cloud Scheduler push notification triggers
- `PORT` — default: 3000

---
> Source: [suphasonnuk/Decode-App](https://github.com/suphasonnuk/Decode-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
