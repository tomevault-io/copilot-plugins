## job-scraper-web

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

JobScout is a full-stack Nuxt 3 web application for job discovery and matching. It's the frontend for a larger ecosystem that includes a Python job scraper and various cloud services. Hosted at https://jobs.timetovalue.org, deployed on Vercel.

## Common Commands

```bash
npm run dev          # Start dev server (HTTPS at https://localhost:3000, requires local certs)
npm run build        # Production build
npx playwright test  # Run E2E tests headless
npx playwright test --debug  # Run E2E tests with UI
npx playwright test tests/login.spec.ts  # Run a single test file
```

## Architecture

**Stack:** Nuxt 3 + Vue 3 Composition API + TypeScript + Pinia + Supabase

### Data Flow

1. **Supabase** is the sole database (PostgreSQL) and auth provider. No ORM — all queries go through the Supabase JS client directly.
2. **`PersistentDataService`** (`services/PersistentDataService.ts`) is a static class that wraps all Supabase database operations. All DB reads/writes should go through this service.
3. **Pinia store** (`stores/jsaStore.ts`) holds global state: `authUser` (Supabase auth session), `dbUser` (user profile from DB), `currentJobs` (job listings), and `selectedUserId` (admin feature).
4. **Server API routes** (`server/api/`) are Nitro endpoints that proxy external service calls (OpenRouter LLM, Mailjet email, Google Cloud Functions, resume upload to GCS). Frontend calls these via `fetch('/api/...')`.

### Key Database Tables (Supabase, schema: `jobscraper`)

- `users` — user profiles and settings
- `users_jobs` — user-job interaction tracking (interest, applied status)
- `recent_high_score_jobs` — view/table of scored job listings
- `user_configs` — key-value user preferences (job_titles, stop_words, skills, etc.)

### Type Definitions

All core data models are in `types/interfaces.ts`: `Job`, `User`, `UserPreferences`, `UserConfig`, `UsersJobs`.

### Authentication

Supabase Auth with email/password + Google OAuth (`nuxt-vue3-google-signin` module). Auth state is checked in page components directly — there is no route middleware for auth guards.

### External Service Integrations

| Service | Purpose | Access Point |
|---------|---------|-------------|
| Supabase | Auth + DB | `utils/supabaseClient.ts` |
| OpenRouter | LLM calls (Gemini, Mistral) | `server/api/openrouter.ts` |
| Mailjet | Email sending | `server/api/sendEmail.ts` |
| Google Cloud Function | Onboarding job generation | `server/api/onboarding.ts` |
| Google Cloud Storage/Vision | Resume upload + OCR | `server/api/upload-resume.ts` |
| Mixpanel | User event analytics | `plugins/mixpanel.ts` |

### Supabase MCP Integration

This project has a Supabase MCP server configured (`.mcp.json`). The database schema is `jobscraper`. Available MCP tools include:

- `mcp__supabase__list_tables` — list tables in a schema (use `["jobscraper"]`)
- `mcp__supabase__execute_sql` — run read queries directly against the database
- `mcp__supabase__apply_migration` — apply DDL changes as tracked migrations
- `mcp__supabase__get_logs` — fetch logs by service (api, postgres, auth, etc.)
- `mcp__supabase__get_advisors` — check for security/performance issues
- `mcp__supabase__generate_typescript_types` — regenerate TS types from the schema
- `mcp__supabase__search_docs` — search Supabase documentation via GraphQL

Use these tools for schema exploration, debugging data issues, and applying migrations instead of connecting to the database manually.

### Development Notes

- Dev server runs over HTTPS using local certificates (`localhost.pem` / `localhost-key.pem`).
- Vue 3 Composition API is used throughout — follow this pattern for new components.
- Component-level state uses `ref`/`reactive`; global state goes in the Pinia store.
- The `composables/` directory holds shared composition functions (e.g., `useUserPreferences`).
- `utils/helpers.ts` contains `transformDataToJobs()` which maps raw Supabase data to typed `Job` objects.
- `utils/llm.ts` has LLM-related utilities including `formatPreferencesAsXml()` for building LLM context.
- Job descriptions are rendered from markdown using the `marked` library.

### Testing

Playwright E2E tests live in `tests/`. The Playwright config auto-starts the dev server and waits for it to be healthy before running tests. Tests run against `https://localhost:3000` with self-signed cert errors ignored.

## Companion Project: job_scraper_py

The Python backend lives at `/Users/davidhague/source/job_scraper_py`. Both projects share the same Supabase database (schema: `jobscraper`).

### How They Connect

```
job_scraper_py (Python worker)    ──scrapes, scores, saves──►  Supabase DB
job_scraper_web (this repo)       ◄──reads jobs, user prefs──  Supabase DB
```

- **Python side** scrapes jobs from 5 boards (Indeed, ZipRecruiter, Glassdoor, LinkedIn, Google), scores them with LLM (16-question assessment across 4 dimensions), generates summaries/requirements, saves to `jobs` + `users_jobs` tables, and sends email notifications via Mailjet.
- **Web side** reads from the `recent_high_score_jobs` view (jobs scored >=50 from the last 14 days or marked interested/applied), displays them to users, and manages user preferences stored in `user_configs`.

### Shared Database Tables

The Python project owns the `jobs` table (with full `description` field) and writes scores/guidance to `users_jobs`. The web app reads the `recent_high_score_jobs` view which joins these. Both projects read/write `users` and `user_configs`.

User preferences set in the web app (`user_configs` keys: `job_titles`, `stop_words`, `skill_words`, `go_words`, `candidate_requirements`, etc.) drive the Python scraper's filtering and scoring logic.

### Python Project Key Files

- `main.py` — orchestration: scrape → clean → match → score → save → email
- `persistent_storage.py` — Supabase client (uses service key, bypasses RLS)
- `llm.py` — unified LLM interface (OpenAI, Anthropic, Gemini)
- `calculate_scores.py` — 16-question weighted scoring via structured LLM output
- `job_scraper.py` — jobspy wrapper with retry logic and deduplication
- `db_scripts/` — SQL schema, tables, views, functions, and RLS policies
- `jobs-app-gcp/` — Google Cloud Function deployment for onboarding

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davehague) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
