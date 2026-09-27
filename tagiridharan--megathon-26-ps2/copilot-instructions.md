## megathon-26-ps2

> This is a static single-page app. `index.html` owns rendering and browser interactions without a frontend framework. The `/api/collections` route is implemented by `netlify/functions/collections.mts` and persists ledger rows through the Drizzle client in `db/index.ts`. Database structure lives in `db/schema.ts`; every schema change requires a generated migration under `netlify/database/migrations`.

# Project guide

## Architecture

This is a static single-page app. `index.html` owns rendering and browser interactions without a frontend framework. The `/api/collections` route is implemented by `netlify/functions/collections.mts` and persists ledger rows through the Drizzle client in `db/index.ts`. Database structure lives in `db/schema.ts`; every schema change requires a generated migration under `netlify/database/migrations`.

## Key directories

- `db/`: Drizzle schema and Netlify Database client.
- `netlify/functions/`: server-side API handlers.
- `netlify/database/migrations/`: deploy-time Postgres migrations.
- `.netlify/`: platform output only; `results.md` is intentionally tracked by the task environment.

## Conventions

Use TypeScript for Netlify Functions and standard Web `Request`/`Response` APIs. Keep database column names in snake_case and JavaScript properties in camelCase. Validate and bound all client input in the function before storing it. Keep rendering states and translations in `index.html` unless the interface grows enough to justify modules.

## Non-obvious decisions

Netlify Database is the canonical shared ledger. `localStorage` holds only unsynced offline submissions, which are retried on reconnect and removed after the API accepts them. The camera scan is a deterministic on-device prototype, not a trained classifier. Role PINs are demo-only and must be replaced with real authentication for production use.

---
> Source: [tagiridharan/megathon-26-ps2](https://github.com/tagiridharan/megathon-26-ps2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
