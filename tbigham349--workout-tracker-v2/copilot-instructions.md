## workout-tracker-v2

> Local-first training log and rehab progress tracker. Successor to the

# Workout Tracker v2

Local-first training log and rehab progress tracker. Successor to the
localStorage app in `..\workout-tracker`. Owner is a physical therapist with a
bilateral knee surgical history; this is both a training log and a rehab
tracker.

## Formatting rule

No em dashes anywhere: code, comments, UI copy, commit messages, docs.

## Stack

React 18 + Vite + Tailwind v4 renderer, Express + better-sqlite3 backend,
Recharts for charts. PWA served by the local Node server so the phone can log
over LAN. All data local; only network calls are to the Peloton API.

## Architecture

Four independent modules with clean interfaces. Keep them decoupled:

- `src/parser/` - freeform notes parser. Pure JS, no UI or DB imports.
  Dependencies are injected (exercise resolver, set history).
- `src/storage/` - SQLite schema, migrations, seed, DAL, JSON export/import.
- `src/analytics/` - progression and energy expenditure. Pure functions.
- `src/integrations/peloton/` - unofficial API adapter plus CSV fallback.

`src/server/` wires them into HTTP. `src/ui/` is the React app.

## Data safety

The DB file lives in `%APPDATA%\workout-tracker-v2\workouts.db` (override
with `WT_DB_PATH`). Logged history is irreplaceable. Never wipe, migrate, or
regenerate it without writing a JSON export first (`exportAll` in
`src/storage/export.js`). Imports commit only through the review flow, in one
transaction.

## Verification

`npm test` (vitest), `npm run build`. Parser changes must keep
`npm run parse:report` accounting for every fixture line with zero silent
drops. Record every architectural choice and parsing heuristic in
`DECISIONS.md`.

## Fixtures

`test/fixtures/workouts-sample.md` is the owner's real log, copied verbatim.
Never edit it. Add new cases as separate fixture files.

---
> Source: [tbigham349/workout-tracker-v2](https://github.com/tbigham349/workout-tracker-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
