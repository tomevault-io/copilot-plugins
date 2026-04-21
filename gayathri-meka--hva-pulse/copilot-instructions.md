## hva-pulse

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev        # Start dev server (http://localhost:3000)
npm run build      # Production build
npm run lint       # ESLint
npm test           # Vitest watch mode
npm run test:run   # Vitest single run (CI)
npx tsx ask-pulse-evals/run-evals.ts   # Ask Pulse eval suite (needs OPENAI_API_KEY + SUPABASE_SERVICE_ROLE_KEY)
```

Tests live in `__tests__/` mirroring the source tree. Mock `next/navigation`'s `redirect` to throw so execution stops as in real Next.js: `vi.fn().mockImplementation((url) => { throw new Error(\`NEXT_REDIRECT:\${url}\`) })`.

## Environment Variables

```
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY       # Required for Supabase Storage uploads (JD files, resumes)
GOOGLE_SERVICE_ACCOUNT_EMAIL
GOOGLE_PRIVATE_KEY              # Multiline PEM key
GOOGLE_SHEET_ID
JOOBLE_API_KEY                  # Required for Job Outreach scraper
OPENAI_API_KEY                  # Required for ask-pulse-evals only (evaluation suite)
ANTHROPIC_API_KEY               # Required for Ask Pulse (claude-sonnet-4-6)
GOOGLE_ALUMNI_SHEET_ID          # Alumni roster Google Sheet ID
MCP_DATABASE_URL                # Read-only Postgres URL for the MCP server (pulse_mcp_ro role)
                                # Format: postgresql://pulse_mcp_ro:[PASSWORD]@db.[PROJECT_REF].supabase.co:5432/postgres
                                # See migrations/018_mcp_readonly_role.sql for setup instructions
```

`SUPABASE_SERVICE_ROLE_KEY` must be added manually to Vercel — it is not a `NEXT_PUBLIC_` var and won't be auto-detected.

`MCP_DATABASE_URL` is only needed locally and in any environment running the MCP server. It does not go to Vercel (the MCP server is not deployed there).

## Architecture

### Route Groups

Two top-level route groups with separate layouts and auth contexts:

- **`app/(protected)/`** — Admin and LF users: dashboard, learners, users, placements. Layout at `app/(protected)/layout.tsx` checks `getAppUser()` and redirects learners to `/learner`.
- **`app/(learner)/learner/`** — Learner-facing placement surface: single Dashboard (role feed + filter pills + PlacementSnapshot), role detail, profile/resume. Layout at `app/(learner)/learner/layout.tsx`.

Middleware (`middleware.ts`) enforces authentication on all protected paths and redirects authenticated users away from `/login`. The `protectedPrefixes` array AND the `matcher` config must both be updated when adding new protected routes.

### Identity & Auth

Dual-layer identity:
1. **Supabase Auth** — Google OAuth session; stores email in auth metadata.
2. **`users` table** — App roles (`admin | LF | learner`) and display name.

`getAppUser()` in `lib/auth.ts` (React `cache`-wrapped) looks up the authenticated user's email in the `users` table to get their app role. Every server component and action calls this.

Role enforcement is done in layouts (redirect) and server actions (`requireAdmin()` helper).

In API route handlers, use `createServerSupabaseClient()` directly for auth — do NOT use `getAppUser()`, as React `cache` does not work in route handlers.

### Server Actions Pattern

All mutations use Next.js Server Actions (`'use server'`). Pattern:

```ts
export async function doSomething(formData: FormData) {
  await requireAdmin()                          // role check + redirect
  const supabase = await createServerSupabaseClient()
  await supabase.from('table').insert(...)
  revalidatePath('/affected/path')              // clear Next.js cache
}
```

All pages that read mutable data use `export const dynamic = 'force-dynamic'`.

### Supabase Clients

- `lib/supabase.ts` — browser client (for client components)
- `lib/supabase-server.ts` — server client with SSR cookie handling (for server components and actions)
- For storage operations requiring elevated privileges, create an admin client directly with `createClient(url, SUPABASE_SERVICE_ROLE_KEY)` — see `uploadJdAttachment()` in `app/(protected)/placements/actions.ts`.

### Placements Module

Admin-facing (`app/(protected)/placements/`) and learner-facing (`app/(learner)/learner/`) share the same Supabase tables:

- `companies` → `roles` → `applications` (cascade delete)
- `resumes` — stored in `resumes` Supabase Storage bucket at `{user_id}/{timestamp}.pdf`
- `jd-files` — stored in `jd-files` Supabase Storage bucket at `{role_id}.pdf`
- `role_preferences` — learner "not interested" signals with reasons

Application status progression: `applied → shortlisted → hired` with two dropout statuses:
- `not_shortlisted` — company did not select for interview (pre-shortlist dropout)
- `rejected` — rejected after interview (post-shortlist dropout)

`applications` also stores `not_shortlisted_reason` and `rejection_feedback` (nullable TEXT). Admin sets these via a required modal when changing to those statuses; they are displayed to the learner on the role detail page.

Company display order is managed via `sort_order` column, with drag-to-reorder in `CompaniesListClient.tsx` (dnd-kit).

### Navigation

`components/NavLinks.tsx` — `ADMIN_LINKS` array drives the sidebar. SVGs are inline HeroIcons outline style.

Active nav state: `bg-zinc-800 text-white` with a left green (`#5BAE5B`) bar indicator.

### Google Sheets Sync

`app/api/sync/route.ts` (POST, admin-only) reads a Google Sheet via service account, upserts learners into Supabase, and deletes rows no longer present in the sheet. LF name-to-user_id mapping is resolved at sync time.

### Alumni Data Ownership — Important

Two separate data sources feed the `alumni` table. Do NOT mix them up:

- **Historical cohorts (pre-2025-26)** — managed entirely via the alumni sheet sync (`/api/sync-alumni`). Company/role/salary all come from the Google Sheet. Re-syncing the sheet is safe; it only touches rows present in the sheet.
- **Current cohort (2025-26 onwards)** — alumni rows are auto-created by the learner sync (step 6) for learners with status "Placed - HVA" or "Placed - Self". Company/role/salary are populated automatically when an admin marks an application as `hired` in the Placements UI (see `updateApplicationStatus` in `actions.ts`). **Do not add these learners to the alumni sheet** — their data lives in Pulse.

The alumni sheet sync will never overwrite Pulse-managed rows because those learners are not in the sheet. The learner sync uses `ignoreDuplicates: true` so it won't overwrite existing alumni rows either.

**Cohort analytics live computation:** Only the cohort defined by `LIVE_COHORT` in `app/(protected)/alumni/page.tsx` is auto-computed from the learners table. All other cohorts use manually entered `cohort_stats`. When a new cohort goes live, update `LIVE_COHORT` in that file.

### Job Outreach Engine

`app/(protected)/placements/` includes a Job Outreach tab backed by:

- DB: `migrations/003_job_outreach.sql` — `job_personas` and `job_opportunities` tables
- Scraper: `lib/scraper.ts` (Jooble API) + API endpoint at `app/api/scrape/route.ts`
- Requires `JOOBLE_API_KEY` env var

### UI Conventions

- Tailwind CSS utility classes throughout; custom green `#5BAE5B` for active nav states
- Tab navs: `border-b` container, active tab indicator = `h-0.5 bg-[#5BAE5B]` absolutely positioned at bottom
- Client components that need optimistic updates use `useTransition` with server actions
- Modals are rendered inline via the `Modal.tsx` component (fixed backdrop + centered panel), not a portal
- TanStack Table v8 is used for the learners, applications, and matching tables with column resizing persisted to `localStorage`
- Data tables: always use TanStack Table v8 (same pattern as LearnersTable / AlumniTable) — column resizing persisted to localStorage, column visibility toggle, multi-select FilterDropdown per column, row count display
- Status badges follow a consistent color scheme: blue=applied/reviewed, amber=shortlisted/in-process, emerald=hired/open, red=rejected, zinc=closed/not-shortlisted/not-interested

## Database Schema

Full schema dump is at `docs/schema.sql`. Always refer to this for column names and types — do not infer from migration files, which are incomplete.

**When adding a new query, always ask: does this column need an index?** Indexes to consider whenever you add a `.eq()`, `.in()`, or `.filter()` on a column that isn't already indexed:
- Any foreign key column used in joins or filters (e.g. `user_id`, `role_id`, `learner_id`)
- Any column used as a filter in a list/analytics page (e.g. `lf_name`, `batch_name`, `status`, `preference`)

Existing indexes are in `migrations/007_performance_indexes.sql` and `migrations/020_missing_indexes.sql`. Add new ones in a new migration file.

**After any schema change**, regenerate the dump and commit it alongside the migration:

```bash
supabase db dump --linked --schema public -f docs/schema.sql
```

**To check if the current dump is up-to-date** (empty diff = current, any output = stale):

```bash
supabase db dump --linked --schema public | diff docs/schema.sql -
```

## Known Issues

- Pre-existing TS error in `.next/types/validator.ts` about `(learner)/learner/my-roles/page.js` — not from our code, safe to ignore
- `npm run lint` broken locally (next lint path issue) — use `npx tsc --noEmit` for type checking instead

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gayathri-meka) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
