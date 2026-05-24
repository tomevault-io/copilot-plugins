## moemeter-api

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev        # Start local dev server at http://localhost:8787 (via wrangler)
npm test           # Run Jest tests (test files live in test/ directory, matched by **/test/**/*.+(ts|tsx))
npm run format     # Prettier format all src/**/*.{ts,js,json}
npm run deploy     # Deploy to Cloudflare Workers via wrangler
```

Environment variables are stored as Wrangler secrets (not in `.env`). For local dev, use `.dev.vars` (gitignored).

## Architecture

This is a **Cloudflare Workers** API (runs on the edge, no Node.js runtime) built with **Hono** and **TypeScript**.

### Data flow

1. **Scraping** (`src/scraping/user.ts`) — fetches raw HTML from `bookmeter.com` and parses user stats via regex
2. **Bookmeter API service** (`src/bookmeter-api/`) — calls the sibling `bookmeter-api` Cloudflare Worker (bound as `env.BOOKMETER_API`) to fetch user reads and reviews
3. **Core logic** (`src/core/`) — orchestrates a full user import: fetch reads from the API service, upsert books/reads/reviews into the DB
4. **DB layer** (`src/db/`) — raw SQL via the `postgres` library (tagged template literals). Each file maps to a domain (users, books, reads, reviews, etc.)
5. **Routes** (`src/routes/`) — thin Hono route handlers; one file per resource, mounted in `src/index.ts`

### Database

- **Supabase (PostgreSQL)** via direct connection string (`DATABASE_URL`) using the `postgres` npm package
- Schema is in `schema.sql`; pull latest with `./schema-pull.sh`
- Several leaderboards are **materialized views** (`ranked_users`, `yearly_leaderboard`, `lonely_leaderboard`, `reading_affinity_leaderboard`) that must be explicitly refreshed after data changes — always call the corresponding `refresh*` functions in `src/db/users.ts` after bulk writes
- Supabase client (`src/db/supabase.ts`) is only used for keep-alive pings, not queries

### Scheduled jobs

Two cron triggers in `wrangler.jsonc`:
- `0 0,3,6,9,12,15,18,21 * * *` — full sync of all users (descending book count)
- `*/3 0,3,6,9,12,15,18,21 * * *` — retry-only sync for users with `sync_status = 'failed'`

Both are handled in the `scheduled()` export in `src/index.ts` which calls `syncAllUsers` from `src/jobs/index.ts`.

### Auth

Protected routes use the `validateGroupAuth` middleware (`src/middlewares/auth.ts`), which requires `group_id` + `password` in the request body verified against the `groups` table.

### Key types

- `AppEnv` (`src/types/app_env.ts`) — Cloudflare Worker bindings: `DATABASE_URL`, `SUPABASE_URL`, `SUPABASE_KEY`, `DEBUG`, `BOOKMETER_API`
- `BookmeterApiService` (`src/types/bookmeter_api_service.ts`) — interface for the sibling Worker service binding
- Domain models are in `src/db/models.ts`: `User`, `Book`, `Read`, `Review`, `Group`

### Bookcase

Some users have a `bookcase` field — a secondary Bookmeter list that supplements their main read count. When present, `original_books_read` tracks the main count and `books_read` reflects the bookcase total.

## Skills

### merge-books (`.claude/merge-books.md`)

Finds and merges duplicate book entries. Trigger phrases: "find duplicate books", "check books that need merging", "merge duplicate books", "run book merge check".

Runs in 5 steps: load exceptions from `book_merge_exceptions` table → exact-dupe SQL query → visual comparison via Explore agent on exported book list → approval → merge via `POST /manual_book_merges`. Never auto-merges. Requires local dev server (`npm run dev`) for merge API calls.

---
> Source: [khaitruong922/moemeter-api](https://github.com/khaitruong922/moemeter-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
