## wc-predictions-with-friends

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev              # Start Vite dev server
npm run build            # Production build (runs lint + typecheck + vite build)
npm run typecheck:strict # TypeScript strict validation only
npm run preview          # Serve built dist

# Linters
npm run lint             # Run all linters
npm run lint:tokens      # Validate CSS custom property tokens
npm run lint:typography  # Typography scale validation
npm run lint:routes      # Check for legacy route references
npm run lint:no-debug-logs # Check for debug console logs
npm run lint:ui-components # Enforce Shadcn/ui primitives

# Tests
npm test                 # Run .test.ts files (Node built-in test runner)

# Data scripts
npm run update-matches      # Fetch latest matches from Football Data API
npm run update-leaderboard  # Rebuild leaderboard snapshot from Firestore
npm run update-snapshots    # Run both above
```

## Architecture Overview

**World Cup predictions league app** — users submit match picks, group stage bracket predictions, and knockout bracket predictions; a live leaderboard tracks standings.

**Stack**: React 18 + TypeScript + Vite, Firebase (Auth + Firestore), Tailwind CSS, React Router v6 (HashRouter for GitHub Pages), Shadcn/ui.

**Theme**: Dark by default. Light mode activated via `.light` class on `<html>`. No system/media-query mode.
**Two surfaces**:
- Default app (`/` routes) — desktop-first
- Mobile companion (`/m/*` routes) — touch-optimized companion view (fully out of scope until explicitly instructed — do not touch any `/m/*` routes)

**Data flow**:
1. Tournament state comes from static JSON snapshots in `public/data/` (matches, members, picks, leaderboard, bracket groups/knockout). These are fetched at runtime with ETag caching.
2. User-specific data (picks, bracket predictions, profile/theme) is stored in and read from Firestore.
3. Firebase Auth gates access; membership is validated against a Firestore email allowlist (`leagues/{leagueId}/members/{email}`).
4. Leaderboard deltas are pre-computed by GitHub Actions — frontend reads only, no real-time listeners.

**Demo mode**: `src/services/demoService.ts` intercepts all service calls. Lives alongside the real services it wraps. Admin-only. Never bleeds into real service logic. Edits live in localStorage. Never writes to Firestore.

## Key Directories

| Path | Purpose |
|------|---------|
| `src/components/ui/` | Shadcn/ui primitives — all UI elements must come from here |
| `src/components/` | Domain/feature components |
| `src/pages/` | Page-level route components |
| `src/hooks/` | Data-fetching and state hooks |
| `src/services/` | All Firebase and external data access |
| `src/lib/` | Pure utilities: scoring, bracket logic, formatters |
| `src/types/` | TypeScript type definitions |
| `src/styles/globals.css` | Design tokens (CSS variables) — sole source of truth for colors |
| `src/styles/shadcn-overrides.css` | Shadcn semantic vars remapped to design tokens |
| `public/data/` | Static JSON snapshots fetched at runtime |
| `scripts/` | Admin utilities: data sync, demo simulation, Firestore seeding |

## Data Flow

1. **Static snapshots** (`public/data/*.json`) — matches, members, picks, leaderboard, scoring config. Fetched at runtime via `fetchSnapshot()` with ETag caching in sessionStorage.
2. **Firestore** — user-specific data: picks, bracket predictions, profile/theme settings. Never read for public/shared data.
3. **Firebase Auth** — Google Sign-In; membership validated against Firestore email allowlist (`leagues/{leagueId}/members/{email}`).
4. **Leaderboard deltas** — pre-computed by GitHub Actions; frontend reads only, no real-time listeners.

## Routing

HashRouter (required for GitHub Pages). All routes use `/#/` prefix.

- `/` — Landing/Play center
- `/results` — Match results
- `/group-stage-picks` — Group stage bracket
- `/bracket` — Knockout bracket
- `/league` — Full league standings
- `/control` — Admin panel (admin only)
- `/m/*` — Mobile companion (out of scope)

## Firestore Structure

```
leagues/{leagueId}/
  members/{normalized_email}   # auth, profile, theme, rivalUserIds, isAdmin
  picks/{userId}               # match predictions array
  bracket-group/{userId}       # group stage bracket predictions
  bracket-knockout/{userId}    # knockout bracket predictions
```

Security rules enforce tournament deadlines:
- Group stage lock: 2026-06-11 18:30 UTC
- First KO kickoff: 2026-06-28 19:00 UTC

## Demo Mode

`src/services/demoService.ts` intercepts all service calls for admin-only demo scenarios. Edits live in localStorage; never writes to Firestore. `fetchSnapshot()` redirects `public/data/` reads to `public/data/demo/scenarios/{scenarioId}/` when demo mode is active.

## Styling Constraints

- No hardcoded hex values anywhere — all colors must use tokens from `src/styles/globals.css`.
- Shadcn variables are remapped in `src/styles/shadcn-overrides.css` — do not redefine inline.
- All UI primitives (buttons, inputs, dialogs, etc.) must use Shadcn/ui from `src/components/ui/`.

## Service Layer Rules

- Components must never import Firebase directly.
- `src/lib/` is for pure utilities only. Anything touching Firebase or external data belongs in `src/services/`.
- `demoService.ts` must never be imported by real service logic.

## Local Dev with Emulators

Add to `.env.local`:
```
VITE_USE_FIREBASE_EMULATORS=true
VITE_FIREBASE_EMULATOR_HOST=127.0.0.1
VITE_FIREBASE_AUTH_EMULATOR_PORT=9099
VITE_FIRESTORE_EMULATOR_PORT=8080
```

Then: `firebase emulators:start` and `npm run dev`.

## CI/CD

- `deploy-pages.yml`: Builds and deploys to GitHub Pages on push to `main`.
- `update-matches.yml`: Scheduled daily at 05:15 UTC to refresh `matches.json` and leaderboard snapshots.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/caharsha91) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
