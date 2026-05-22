## upworktonotion

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.


## Commands

```bash
npm install       # install dependencies
npm run dev       # start local dev server (http://localhost:3000)
npm run build     # production build
npm run lint      # run ESLint
npm run test      # run Vitest once
npm run test:watch # run Vitest in watch mode
```

## daisyUI reference
Use this for all UI components: @docs/llms.txt

## Architecture

**Stack:** Next.js 16 (Pages Router) + TypeScript, deployed serverless on Vercel. Tests via Vitest.

**Flow:** cron-job.org (1 min) → `/api/sync` → Upwork API → Zod validation → Supabase log → Notion upsert → Pino logs

### Key files

| Path | Purpose |
|------|---------|
| `src/pages/api/sync.ts` | Main sync entry point (GET only; triggered every 1 min by cron-job.org) |
| `src/pages/api/upwork/auth.ts` | Starts Upwork OAuth2 flow — redirects user to Upwork |
| `src/pages/api/upwork/callback.ts` | Receives OAuth code, exchanges for tokens with retry logic, saves to Supabase |
| `src/pages/api/upwork/fetch.ts` | Calls Upwork REST API via `callUpwork` helper |
| `src/pages/api/upwork/gql.ts` | Proxy to Upwork GraphQL endpoint (`https://api.upwork.com/graphql`) |
| `src/lib/upworkToken.ts` | OAuth token lifecycle: load from Supabase, auto-refresh when <2 min left |
| `src/lib/upworkClient.ts` | `callUpwork()` — authenticated REST wrapper using `getValidAccessToken` |
| `src/lib/notion.ts` | `upsertToNotion()` — find-or-create Notion pages keyed on `External ID` |
| `src/lib/supabase.ts` | `getSupabase()` — lazy-init Supabase client (service role, no session persistence) |
| `src/lib/upwork.ts` | Zod schema for `UpworkItem`; `fetchUpworkItems()` via Upwork GraphQL API |
| `src/lib/requireAuth.ts` | `requireAuth()` — API route guard checking `Authorization: Bearer <API_SECRET>` |
| `src/lib/logger.ts` | Pino logger; pretty-prints in dev, JSON in prod |

### OAuth token storage

Upwork OAuth tokens are stored **per user** in the Supabase `upwork_tokens` table, keyed by `user_id`. `getValidAccessToken()` transparently refreshes when expiry is within 2 minutes.

### Notion upsert

`upsertToNotion()` queries the Notion database by the `External ID` rich-text property to detect existing pages, then either updates or creates. Uses `notion.request()` directly for the query to stay compatible with SDK v5.

### Notion databases

The app uses **3 separate Notion databases**. Each must be shared with the Notion integration (open the DB → top-right `...` → **Connections** → add your integration).

Get a database ID from its URL: `https://notion.so/workspace/`**`<32-char-hex>`**`?v=...`

#### 1. Filtered Job Feed (`NOTION_JOB_FEED_DATABASE_ID`)

Jobs fetched from Upwork marketplace matching your filters. App writes here.

| Property | Type | Notes |
|----------|------|-------|
| `Name` | Title | Job title |
| `Description` | Rich text | Truncated to 2000 chars |
| `External ID` | Rich text | `job-<upwork_id>` — dedup key |
| `Client` | Rich text | Client's country |
| `Value` | Number | Max hourly rate or fixed amount (USD) |
| `Job Type` | Select | `Hourly` or `Fixed` |
| `Applied` | Checkbox | True if you submitted a proposal |
| `Proposal link` | URL | Link to your proposal (when Applied = true) |
| `Upwork Link` | URL | Job posting URL |
| `Created` | Date | Published date |

#### 2. Job Feed Filters (`NOTION_JOB_FILTERS_DATABASE_ID`)

Each row is a saved search. App reads this to know what to fetch from Upwork.

| Property | Type | Notes |
|----------|------|-------|
| `Name` | Title | Label for the filter (e.g. "Figma UI") |
| `Active` | Checkbox | Uncheck to pause this filter |
| `Skill Expression` | Rich text | Free-text skill query (e.g. `UX/UI`) |
| `Category` | Multi-select | e.g. `Design & Creative`, `Web / Mobile & Software Dev` (12 options) |
| `Subcategory` | Multi-select | e.g. `Design › Product Design` (70 options with category prefix) |
| `Job Type` | Select | `Hourly` or `Fixed` |
| `Min Budget` | Number | Min hourly rate or fixed price (USD) |
| `Max Budget` | Number | Max hourly rate or fixed price (USD) |
| `Experience Level` | Multi-select | `Expert`, `Intermediate`, `Entry` — only first value used |
| `Verified Payment Only` | Checkbox | Only clients with verified payment method |
| `Duration` | Multi-select | `Week`, `Month`, `Quarter`, `Semester`, `Ongoing` |
| `Workload` | Select | `Full Time`, `Part Time`, `As Needed` |
| `Days Posted` | Number | Max days since posting |
| `Max Proposals` | Number | Upper bound on proposal count |
| `Min Client Hires` | Number | Minimum prior hires by client |
| `Min Client Rating` | Number | Minimum client feedback score |
| `Previous Clients Only` | Checkbox | Only clients you've worked with before |
| `Enterprise Only` | Checkbox | Enterprise clients only |

#### 3. Work Diary (`NOTION_DIARY_DATABASE_ID`)

One row per contract per work day. App writes here.

| Property | Type | Notes |
|----------|------|-------|
| `Name` | Title | Week label (e.g. `Week 15`) |
| `ID` | Rich text | `contract-<id>-<yyyyMMdd>` — dedup key |
| `Contract name` | Rich text | Contract title from Upwork |
| `Date` | Date | Work day (ISO format) |
| `Minutes` | Number | Tracked minutes (cells × 10) |
| `Rate` | Number | Hourly rate (USD), if applicable |

#### Setting the env vars

**.env.local:**
```
NOTION_JOB_FEED_DATABASE_ID=<id>
NOTION_JOB_FILTERS_DATABASE_ID=<id>
NOTION_DIARY_DATABASE_ID=<id>
```

**Vercel (production):**
```bash
npx vercel env add NOTION_JOB_FEED_DATABASE_ID production
npx vercel env add NOTION_JOB_FILTERS_DATABASE_ID production
npx vercel env add NOTION_DIARY_DATABASE_ID production
npx vercel --prod
```

### Upwork API access

- REST calls go through `callUpwork(path)` in `src/lib/upworkClient.ts`.
- GraphQL calls are proxied through `/api/upwork/gql` (POST `{query, variables}`).
- Base URL for REST: `https://www.upwork.com/api/v3/`.
- The sync pipeline uses `vendorProposals` GraphQL query directly (not the proxy). Page size must be ≤ 40.

## Environment variables

| Variable | Used by |
|----------|---------|
| `NOTION_TOKEN` | Notion client auth |
| `NOTION_JOB_FEED_DATABASE_ID` | Job feed output DB |
| `NOTION_JOB_FILTERS_DATABASE_ID` | Filter config DB (read by app) |
| `NOTION_DIARY_DATABASE_ID` | Work diary output DB |
| `SUPABASE_URL` | Supabase client |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase client |
| `UPWORK_CLIENT_ID` | OAuth flow |
| `UPWORK_CLIENT_SECRET` | OAuth token exchange & refresh |
| `UPWORK_REDIRECT_URI` | OAuth callback URL |
| `UPWORK_PERSON_ID` | Freelancer's numeric Upwork user ID — used by `fetchContractDays()` to query work diary |
| `LOG_LEVEL` | Pino log level (default: `info`) |
| `API_SECRET` | Auth for protected API routes (`Authorization: Bearer <secret>`) |

## Testing after changes

After every incremental change, verify:

### API routes

```bash
# Health check — should always work, no env vars needed
curl -s http://localhost:3000/api/ping | jq .
# expect: {"ok":true,"service":"UpworkToNotion","version":"v0.1"}

# Notion connectivity (requires NOTION_TOKEN + NOTION_JOB_FEED_DATABASE_ID)
curl -s http://localhost:3000/api/notion-debug | jq .
# expect: {"ok":true,"title":"..."}

# Sync endpoint (requires API_SECRET + Notion + Supabase env vars)
curl -s -H "Authorization: Bearer $API_SECRET" http://localhost:3000/api/sync | jq .
# expect: {"ok":true,"jobs":{"fetched":...,"created":...,"updated":...,"skipped":...},"contracts":{...},"durationMs":...}

# Upwork fetch (requires API_SECRET + valid OAuth tokens in Supabase)
curl -s -H "Authorization: Bearer $API_SECRET" \
  'http://localhost:3000/api/upwork/fetch?path=contracts?limit=5' | jq .
# expect: {"ok":true,"url":"...","data":{...}}

# Upwork GraphQL proxy (requires API_SECRET + valid OAuth tokens)
curl -s -X POST http://localhost:3000/api/upwork/gql \
  -H "Authorization: Bearer $API_SECRET" \
  -H 'Content-Type: application/json' \
  -d '{"query":"{ user { id name } }"}' | jq .
# expect: {"ok":true,"status":200,"data":{...}}

# Seed demo rows into Notion (requires API_SECRET + Notion env vars)
curl -s -H "Authorization: Bearer $API_SECRET" \
  'http://localhost:3000/api/notion/seed-applied?count=2' | jq .
# expect: {"ok":true,"created":2,"results":[...]}
```

### Build, lint & test

```bash
npm run test    # must exit 0 — runs Vitest
npm run build   # must exit 0 — catches type errors
npm run lint    # must exit 0
```

### OAuth flow

1. Open `http://localhost:3000/api/upwork/auth` in a browser — should redirect to Upwork.
2. After authorizing, Upwork redirects to the callback URL. Check the response for `{"ok":true,"saved":true}`.
3. Verify token was stored: query the `upwork_tokens` table in Supabase.

### Checklist for every change

1. `npm run test` passes
2. `npm run build` passes
3. `npm run lint` passes
4. `curl /api/ping` returns `{"ok":true}`
5. If you touched Notion code: `curl -H "Authorization: Bearer $API_SECRET" /api/sync` succeeds
6. If you touched Upwork code: `curl -H "Authorization: Bearer $API_SECRET" /api/upwork/fetch` returns data
7. If you touched OAuth code: run the full auth flow above

## PR requirements

Every PR body must include a spec link matching `specs/[0-9]{4}-` — the `spec-check` CI job enforces this. Use the PR template in `.github/pull_request_template.md`.

## Spec

The product spec lives in `specs/specs/0001-upwork-notion-v0.1.md`. The sync pipeline is fully wired up and working end-to-end.

## Current status (as of 2026-04-28)

### What's done

- Full OAuth flow working (auth → callback → tokens saved to Supabase)
- Upwork GraphQL schema discovered via `/api/upwork/gql-introspect`
- Sync pipeline: three parallel tracks per cron run:
  1. **Proposals** (`fetchUpworkItems`): fetches pending/active/hired proposals, used for cross-referencing job feed items with submitted proposals
  2. **Job feed** (`fetchJobFeed`): reads active filters from Notion filters DB, runs one query per filter, deduplicates by job ID, writes to job feed Notion DB. 10 jobs per filter query (no pagination on this endpoint). Annotates jobs with proposal URL when already applied.
  3. **Work diary** (`fetchContractDays`): 3-step approach — writes one row per contract per day to `NOTION_DIARY_DATABASE_ID`
- `contractList` / `vendorContracts` permanently blocked (Upwork partner API scope). Workaround: use `talentWorkHistory` for active contract IDs.
- **3 Notion databases** wired up (env vars set locally + Vercel):
  - `NOTION_JOB_FEED_DATABASE_ID` — filtered job results (output)
  - `NOTION_JOB_FILTERS_DATABASE_ID` — saved searches (input, read by app)
  - `NOTION_DIARY_DATABASE_ID` — per-day work diary rows
- Notion client pinned to API version `2022-06-28` (SDK default `2025-09-03` removed the `databases/query` endpoint)
- Job feed filters: human-readable multi-select labels in Notion → numeric Upwork IDs via `CATEGORY_ID_MAP` / `SUBCATEGORY_ID_MAP` in `upwork.ts`. 12 filter fields supported (skill, category, subcategory, job type, budget, experience level, verified payment, duration, workload, proposals cap, client hires/rating, flags)
- Sync runs every 1 minute via cron-job.org (external cron hitting `/api/sync`). GitHub Actions schedule disabled. Vercel has no cron config.
- Hourly job rates stored as separate `Rate Min` and `Rate Max` Number fields in Notion Job Feed DB. `Value` field is fixed-price jobs only.
- Budget filter uses range overlap: keep hourly jobs where `rateMax >= filter.minBudget` AND `rateMin <= filter.maxBudget`. For fixed jobs, filter against `value` directly.
- Dashboard auto-refreshes settings every 15 seconds; last sync counter ticks in real time (seconds).
- API throttling: proposals fetched once per hour, work diary once per 10 minutes (timestamps stored in `last_proposals_sync_at`, `last_diary_sync_at` in `user_settings`). Required `NOTIFY pgrst, 'reload schema'` in Supabase after ALTER TABLE.
- `prev_sync_at` stored in `user_settings` to compute actual sync interval (shown in seconds on dashboard).
- Dashboard shows "Synced at HH:MM" toast after each cron-triggered sync (detected via polling `last_sync_at`).
- Upwork job URLs fixed: use `ciphertext` field from GraphQL (e.g. `~022048894189896190929`), NOT numeric `id`. Numeric IDs return 404.
- Singleton token fallback removed from `getValidAccessToken` (security fix). Tokens are strictly per-user row.
- Cron interval changed to 2 minutes at cron-job.org to prevent overlapping syncs causing duplicate Notion pages.
- `filters_db_id` column dropped from Supabase `user_settings` table and removed from all code — was never read by the sync pipeline (filters DB comes from env var `NOTION_JOB_FILTERS_DATABASE_ID`).

### Web UI (as of 2026-04-28)

- **Settings page** (`/settings`): full-width `tabs tabs-box` daisyUI component. Two tabs: Upwork and Notion.
  - Upwork connected state: inline SVG logo + "Connected" badge + green circle reconnect button (tooltip: "Reconnect Upwork API"). Plain inputs, full-width `btn-soft btn-primary` Save button.
  - Upwork not-connected state: daisyUI `alert alert-vertical sm:alert-horizontal` instructions box, plain inputs, single "Save & Connect" button that saves credentials then immediately redirects to `/api/upwork/auth`.
  - Notion tab: same pattern. Notion logo from Figma (node 129:8161). Fields: Integration token, Job Feed DB ID, Work Diary DB ID (filters DB ID removed).
  - Both tabs: skeleton loading state while settings fetch.
- **Profile page** (`/profile`): Personal info (first name saved to `user_metadata`, read-only email), Change password, Danger zone with delete account modal. Delete uses `DELETE /api/user/delete` (service role admin delete). Full-width card layout matching settings page.
- **Dashboard** (`/dashboard`): Non-connected state replaced with full-width banner from Figma (node 140:8343). Background image (`/figma/dashboard-banner-616c12.png`) on the right. Stacks vertically on small screens (image on top, divider, text below). Shadow + rounded-[17px].
- **Sidebar**: gradient `linear-gradient(to bottom, #312E81, #0F0E1A)` replacing flat `#2F4F82`.
- **Figma assets**: downloaded to `public/figma/` — `upwork-logo.svg`, `notion-logo.svg`, `dashboard-banner-616c12.png`. Upwork and Notion logos are inline SVG components in `settings.tsx`.
- **Filters page**: "Verified Payment only" toggle uses daisyUI toggle with checkmark (enabled) / X (disabled) SVG icons.

### Notion Job Feed DB — updated schema

| Property | Type | Notes |
|----------|------|-------|
| `Name` | Title | Job title |
| `Description` | Rich text | Truncated to 2000 chars |
| `External ID` | Rich text | `job-<upwork_id>` — dedup key |
| `Client` | Rich text | Client's country |
| `Value` | Number | Fixed-price amount only (USD). Empty for hourly jobs. |
| `Rate Min` | Number | Hourly job minimum rate (USD). Empty for fixed jobs. |
| `Rate Max` | Number | Hourly job maximum rate (USD). Empty for fixed jobs. |
| `Job Type` | Select | `Hourly` or `Fixed` |
| `Workload` | Select | Raw `engagement` string from API (e.g. "Less than 30 hrs/week"). Populated when API returns it (~50% of jobs). Use for manual Notion filtering only — not filtered in pipeline. |
| `Applied` | Checkbox | True if you submitted a proposal |
| `Proposal link` | URL | Link to your proposal (when Applied = true) |
| `Upwork Link` | URL | Job posting URL |
| `Created` | Date | Published date |

### Contracts sync — 3-step work diary approach

`fetchContractDays()` in `src/lib/upwork.ts`:
1. `talentWorkHistory(filter: { personId: $UPWORK_PERSON_ID, status: [ACTIVE] })` → active contract IDs + titles + rates
2. Batched `workDays` queries → days with tracked activity this week (Mon–Sun UTC, yyyyMMdd format)
3. Batched `workDiaryContract` queries (up to 10 per request) → count `workDiaryTimeCells` (each = 10 min) → stored as `minutes`

**User ID**: `540749103839944704` (Alexey, stored as `UPWORK_PERSON_ID`)

### Known quirks

- `vendorProposals` pagination limit is 40 (`first: 41+` returns VJCA-6 error)
- `marketplaceJobPostingsSearch` has no pagination — always returns 10 results per query, no `pagination` or `paginationInput` argument exists. This is a hard cap with no workaround at this API tier.
- `marketplaceJobPostingsSearch` only accepts 3 arguments: `marketPlaceJobFilter`, `searchType`, `sortAttributes`. Confirmed via schema introspection.
- Notion SDK v5 ships with API version `2025-09-03` which removed `databases/query` — must pass `notionVersion: "2022-06-28"` when creating the client
- Upwork OAuth scopes are configured at app level in developer portal, not via `scope` param in auth URL — passing `scope` returns `invalid_scope` error
- `workDays` / `workDiaryContract` date format: `yyyyMMdd` (not ISO)
- Weekly earnings blocked — requires Payments scope (`transactionHistory` returns "Authorization failed")
- Category multi-select names cannot contain commas — `"Web / Mobile & Software Dev"` used instead of `"Web, Mobile & Software Dev"`
- `Experience Level` is multi-select in Notion but Upwork API only accepts one value — only the first selected option is used
- `workload_eq` filter (`FULL_TIME` / `PART_TIME`) on `marketplaceJobPostingsSearch` returns 0 results — removed from the pipeline. Workload filter is not exposed in the UI. The `engagement` field on job nodes also returns `null` when `workload_eq` is active. Revisit if Upwork grants a higher API tier.
- `budgetRange_eq` on `marketplaceJobPostingsSearch` is completely non-functional — confirmed by testing with `{ rangeStart: 100, rangeEnd: 200 }` which returned jobs with $3–$5 and $10–$20 rates. The filter is silently ignored regardless of values. Same for `hourlyRate_eq`. Budget filtering is done entirely client-side.
- Hourly jobs have a rate range (min–max), not a single value. `hourlyBudgetMin` and `hourlyBudgetMax` are separate fields. Budget filter uses overlap logic: job is kept if its rate range intersects with the filter range.
- If more than 10 matching jobs are posted between syncs, the extras are permanently missed (no historical fetch possible). Mitigation: sync every 1 minute via cron-job.org.
- Upwork type-level GraphQL introspection is restricted — `__type(name: "...")` returns null for most types. Only `__schema { types { name } }` works to list type names.

### Sync infrastructure

- **Trigger**: cron-job.org, every 1 minute, calls `https://upwork-to-notion.vercel.app/api/sync` with `Authorization: Bearer <API_SECRET>` header
- **GitHub Actions** (`sync.yml`): schedule disabled, `workflow_dispatch` kept for manual runs
- **Vercel**: no cron configured (`vercel.json` has no `crons` key)
- Sync takes ~4 seconds per run. 1-minute interval has ample headroom.
- Overlapping syncs are safe — Notion upserts are idempotent (dedup by `External ID`)

### What's next (Phase 3)

See `docs/progress.md` for the full roadmap. Summary:
- Auto-create Notion databases (remove manual setup)
- Onboarding flow for new users
- Email notifications when new matching jobs appear
- Freelancer profile snapshot (JSS, earnings, top-rated) → Notion


## Coding standards

1. Use latest versions of libraries and idiomatic approaches as of today
2. Keep it simple - NEVER over-engineer, ALWAYS simplify, NO unnecessary defensive programming. No extra features - focus on simplicity.
3. Be concise. Keep README minimal.

---
> Source: [alexey72882/UpworkToNotion](https://github.com/alexey72882/UpworkToNotion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
