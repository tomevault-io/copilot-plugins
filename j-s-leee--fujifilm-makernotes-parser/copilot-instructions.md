## fujifilm-makernotes-parser

> Fujifilm film simulation recipe sharing platform built with Next.js 15 (App Router) + Supabase + Vercel.

# Project: film-simulation.site

Fujifilm film simulation recipe sharing platform built with Next.js 15 (App Router) + Supabase + Vercel.

## Tech Stack

- **Framework**: Next.js 15 (App Router, Server Components)
- **DB**: Supabase (Postgres), RLS enabled on all tables
- **Storage**: Cloudflare R2 (`NEXT_PUBLIC_R2_PUBLIC_URL`)
- **Hosting**: Vercel (Hobby Plan)
- **Language**: TypeScript, Tailwind CSS, shadcn/ui

## Project Structure

```
app/                  # Next.js App Router pages
components/           # React components
lib/                  # Utilities, Supabase clients, queries
fujifilm/             # Fujifilm-specific constants (simulations, cameras)
supabase/migrations/  # SQL migration files (timestamp-ordered)
docs/plans/           # Design & implementation plans
```

## Database

- Two Supabase projects: `project-prod` (production), `project-dev` (development)
- All tables have RLS enabled
- `recipes_with_stats` is the main read VIEW (joins recipes + profiles + lookups)
- Queries use `createStaticClient()` for public reads, `createClient()` for authenticated

## Migration Rules (CRITICAL)

When writing SQL migrations in `supabase/migrations/`, always follow these rules to ensure zero-downtime deployments.

### File Naming

`YYYYMMDDHHMMSS_description.sql` (e.g., `20260311000000_collections.sql`)

### Safe Operations (do freely)

- `CREATE TABLE` — new tables never break existing code
- `ALTER TABLE ADD COLUMN ... DEFAULT ...` — always include DEFAULT for new columns
- `CREATE INDEX CONCURRENTLY` — use CONCURRENTLY to avoid table locks
- `CREATE OR REPLACE VIEW` — view changes are safe
- `CREATE OR REPLACE FUNCTION` — function changes are safe
- `CREATE POLICY` — new RLS policies are safe
- `CREATE TRIGGER` — new triggers are safe

### Dangerous Operations (require 2-step deploy)

Never do these in a single migration if the column/table is actively used by the app:

| Operation | Why Dangerous | How to Do Safely |
|-----------|---------------|------------------|
| `DROP COLUMN` | Running code still references it | Step 1: Remove code references + deploy. Step 2: DROP in next migration |
| `RENAME COLUMN` | Running code uses old name | Step 1: Add new column + copy data. Step 2: Switch code. Step 3: Drop old column |
| `ALTER COLUMN TYPE` | May fail or lock table | Add new column, migrate data, switch code, drop old |
| `ALTER COLUMN SET NOT NULL` | Existing rows may violate | Add DEFAULT first, backfill NULLs, then add constraint |
| `DROP TABLE` | Running code references it | Remove all code references first, drop in next migration |
| `DROP VIEW CASCADE` | Breaks dependent functions | Recreate all dependents in same migration (see soft_delete migration for example) |

### Migration Style

Follow the existing pattern in the codebase:

```sql
-- ============================================================
-- SECTION HEADER: Brief description
-- ============================================================
CREATE TABLE public.example (
  id         bigint PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  user_id    uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  name       text NOT NULL,
  created_at timestamptz DEFAULT now()
);

CREATE INDEX example_user_id_idx ON public.example (user_id);

ALTER TABLE public.example ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Descriptive policy name"
  ON public.example FOR SELECT
  TO anon, authenticated
  USING (true);
```

### Deployment Order

Production deploy is always: **DB first, code second.**

1. Apply migration to prod DB (`supabase db push`)
2. Verify migration success
3. Merge PR to main (triggers Vercel deploy)

### RLS Conventions

- All tables MUST have RLS enabled
- Use `(SELECT auth.uid())` (not `auth.uid()`) in policies for performance
- Policy names should be descriptive English strings
- Common pattern: anon+authenticated for SELECT, authenticated-only for INSERT/UPDATE/DELETE

## Code Conventions

- Use `GALLERY_SELECT` / `RECIPE_DETAIL_SELECT` from `lib/queries.ts` for consistent field selection
- R2 URLs: `process.env.NEXT_PUBLIC_R2_PUBLIC_URL + "/" + path`
- Thumbnails: check `thumbnail_width` to decide if `getThumbnailUrl()` transform is needed
- Prefer parallel data fetching with `Promise.all()` in server components
- Pages export `revalidate = 60` for ISR where appropriate

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/j-s-leee)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/j-s-leee)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
