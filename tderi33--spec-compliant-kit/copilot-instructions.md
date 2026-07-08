## 40-supabase-lovable-cloud-migrations

> How Supabase works under Lovable Cloud — migrations, grants, edge-function deploy, secrets, and the bundler. Read before writing ANY migration or edge function so it actually works once Lovable applies/deploys it.


# Supabase on Lovable Cloud — migrations, edge functions & secrets

This project's backend is a Supabase instance managed by **Lovable Cloud**. Migrations are written here (Cursor) but executed by Lovable's migration runner against the hosted DB. There is **no Supabase dashboard access**, no `SUPABASE_ACCESS_TOKEN`, no `SUPABASE_DB_PASSWORD`, and no local `supabase db push`. If a migration depends on dashboard-only state or missing privileges, it silently breaks the app at runtime (PostgREST returns permission errors and the frontend looks "empty").

Follow these rules so every migration you author here works the first time Lovable applies it.

## 1. The non-obvious rule: `public` schema has NO default Data API grants

Lovable Cloud's PostgREST does **not** inherit default privileges on `public` for `anon`, `authenticated`, or `service_role`. Enabling RLS is **not enough** — without an explicit `GRANT`, every request returns:

```
permission denied for table <name>
```

Every `CREATE TABLE public.<x>` migration MUST include `GRANT` statements **in the same migration**, in this exact order:

```sql
-- 1. CREATE
CREATE TABLE public.<table> (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  -- ... domain columns ...
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

-- 2. GRANT (REQUIRED — match the roles your policies allow)
GRANT SELECT, INSERT, UPDATE, DELETE ON public.<table> TO authenticated;
GRANT ALL ON public.<table> TO service_role;
-- GRANT SELECT ON public.<table> TO anon;   -- ONLY if an anon policy exists

-- 3. RLS
ALTER TABLE public.<table> ENABLE ROW LEVEL SECURITY;

-- 4. POLICIES
CREATE POLICY "..." ON public.<table> FOR SELECT TO authenticated USING (...);
```

Rules of thumb for the grant block:
- **Always** grant `service_role` for tables touched by edge functions, CRON jobs, or admin code (service_role bypasses RLS but still needs the table-level GRANT).
- **Drop `anon`** when every policy scopes to `auth.uid()`. This repo is authenticated-only except for the `feedback_*` guest-iframe path.
- Widen `anon` privileges only for fully public tables (none currently exist outside the `feedback-qr` storage bucket).

## 2. Functions — `EXECUTE` is also not automatic

For `SECURITY DEFINER` RPCs callable from the client (`supabase.rpc(...)` or `supabase.functions.invoke` patterns that hit PostgREST):

```sql
CREATE OR REPLACE FUNCTION public.my_rpc(...) RETURNS ...
LANGUAGE sql STABLE SECURITY DEFINER
SET search_path = public
AS $$ ... $$;

REVOKE ALL ON FUNCTION public.my_rpc(...) FROM PUBLIC, anon;
GRANT EXECUTE ON FUNCTION public.my_rpc(...) TO authenticated, service_role;
-- Only add `anon` if the function is part of a documented public surface
-- (in this repo: feedback_resolve_location, feedback_submit_response, feedback_iframe_completion).
```

Linter findings about "function callable by anon" are real — `REVOKE EXECUTE ... FROM anon` unless the function is intentionally public.

## 3. RLS-enabled tables with zero policies

A table with `ENABLE ROW LEVEL SECURITY` and no policies blocks all client access — that's often the intent for tables only touched by `service_role` edge functions (e.g. `livereach_nvr_token_cache`). The linter flags this as INFO. Declare intent explicitly:

```sql
CREATE POLICY "no client access" ON public.<table>
  FOR ALL TO anon, authenticated
  USING (false) WITH CHECK (false);
```

Service role still bypasses RLS, so edge functions keep working.

## 4. Storage buckets

Bucket policies live in `storage.objects` and follow the same GRANT model. A `public = true` bucket still serves files by direct URL anonymously — but if you also create a permissive `SELECT` policy granting `anon`, you enable **listing/enumeration** of every object. For QR-code-style buckets, scope the listing policy to `authenticated` only:

```sql
DROP POLICY IF EXISTS "Public can read X" ON storage.objects;
CREATE POLICY "Authenticated can list X" ON storage.objects
  FOR SELECT TO authenticated
  USING (bucket_id = 'X');
```

## 5. Roles & RBAC (project-specific)

- `app_role` enum is **2 values: `'admin', 'manager'`**. Do NOT add `attendant`, `employee`, or `user`. Floor attendants are not Supabase-auth users.
- Roles live in `public.user_roles` (never on `user_profiles`).
- RLS uses `SECURITY DEFINER` helpers `public.has_role(uid, role)` and `public.get_user_location(uid)` / `public.get_user_allowed_locations(uid)`. Reuse them — don't inline `EXISTS (SELECT 1 FROM user_roles ...)` in policies (causes recursion).
- Pattern: "admin sees all, manager sees own location."

## 6. Forbidden in migrations

- `ALTER DATABASE postgres ...` — rejected by Lovable's runner.
- Editing `auth`, `storage`, `realtime`, `supabase_functions`, `vault` schemas (except writing policies on `storage.objects`). No triggers on `auth.users` — use the existing `handle_new_user()` trigger pattern.
- Editing shipped migrations. Migrations are **forward-only**; write a new file.
- Editing `src/integrations/supabase/client.ts`, `src/integrations/supabase/types.ts`, `.env` (`VITE_SUPABASE_*`), or `supabase/config.toml` (only `project_id` lives there).
- `CHECK` constraints that reference `now()` or other mutable functions — use a `BEFORE INSERT/UPDATE` trigger instead. Postgres requires CHECK expressions to be immutable, and mutable ones break `pg_dump` restores.

## 7. Timestamps & updated_at

Every user-data table gets `created_at` + `updated_at` (`timestamptz NOT NULL DEFAULT now()`) plus the standard trigger. Reuse the existing function:

```sql
CREATE TRIGGER update_<table>_updated_at
  BEFORE UPDATE ON public.<table>
  FOR EACH ROW EXECUTE FUNCTION public.update_updated_at_column();
```

`public.update_updated_at_column()` already exists — don't redefine it.

## 8. Time/timezone (project hard rule)

All day boundaries are **ET (`America/New_York`), DST-aware**. In SQL, use the existing helper `public.et_day_range_utc(from_et date, to_et date)` to convert ET date ranges to UTC timestamptz windows. Never do naive UTC math like `now()::date` or `current_date` for business-day logic.

## 9. Locations are the canonical site table

`public.locations` already exists and carries every external ID (`flexwash_id`, `connecteam_id`, `livereach_id`, `nvr_*`, `timezone`, lat/long). **Never create a `site` table.** Foreign-key with `location_id uuid REFERENCES public.locations(id)`.

## 10. Verifying a migration before handing it off

Before committing a migration, mentally walk this checklist:

- [ ] Every new `public` table has a GRANT block matching its policies.
- [ ] Every new `SECURITY DEFINER` function has `SET search_path = public` and explicit `GRANT EXECUTE` / `REVOKE FROM anon`.
- [ ] Every RLS-enabled table either has policies or an explicit "no client access" deny policy.
- [ ] No CHECK constraint references a mutable function.
- [ ] No edits to forbidden schemas/files.
- [ ] FK to `auth.users` is via a `profiles`-style table, not direct from a domain table.
- [ ] Time logic uses `et_day_range_utc` / explicit ET conversion.
- [ ] If anything required user-provided secrets (access token, DB password), stop — those don't exist on Lovable Cloud; the Lovable agent has to apply the migration through its `supabase--migration` tool.

## 11. What Cursor cannot do here

- Cannot run `supabase db push`, `supabase link`, or `supabase migration up` — there is no access token or DB password issued for this project. The GitHub Actions `supabase-linter.yml` workflow is non-functional for the same reason.
- Cannot regenerate `src/integrations/supabase/types.ts` — Lovable regenerates it after each applied migration.
- Cannot apply migrations directly. The workflow is: write the `.sql` file in `supabase/migrations/` here, then have Lovable apply it (Lovable's agent calls `supabase--migration` which prompts the user for approval and runs it server-side).

When in doubt, mirror the most recent migration in `supabase/migrations/` — it reflects the current working pattern Lovable's runner accepts.

## 12. Edge functions, secrets & deploy (confirmed June 2026)

Edge functions are **deployed by Lovable from the connected branch (`main`)** — Cursor cannot deploy them. The same "write here, Lovable applies" model as migrations. Concrete constraints learned in production:

### Bundler: own dir + `_shared/` only
The edge-function bundler includes **only the function's own directory plus `supabase/functions/_shared/`**. A **cross-function import** (e.g. `amenities-board` importing `../amenities-attendant-session/_lib.ts`) **builds locally and passes `deno test`, but FAILS at deploy.** So any code shared between functions MUST live in `_shared/` (e.g. `_shared/floorMap.ts`, `_shared/attendantAuth.ts`). Never import one function's `_lib.ts` from another function, and never "fix" a deploy failure by duplicating the helper — move it to `_shared/`.

### Secrets
- **Custom secrets cannot start with `SUPABASE_`** (reserved). Use a project-prefixed name (e.g. `AMENITIES_TABLET_JWT_SECRET`, `AMENITIES_DEVICE_KEY_PEPPER`).
- The auto-injected vars are `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `SUPABASE_DB_URL`. **The project JWT secret is NOT injected and NOT readable** on Lovable Cloud (no dashboard), and **custom JWT issuers / JWKS / third-party-auth registration are not exposable.** So you **cannot mint a token that native PostgREST/Realtime will accept**, and you must **not** rotate the project JWT secret to work around it (destructive: logs out every user, rotates the anon/service keys). If a flow needs a non-OAuth bearer (devices, the attendant tablet), **sign your own token with a project-prefixed secret and verify it inside your own edge functions** (validate-then-service-role; see `_shared/deviceAuth.ts` and `_shared/attendantAuth.ts`). See `10-amenities-module.mdc` for the tablet-token specifics.
- Realtime: native authorized Realtime/RLS for a non-OAuth client is **not reachable** here; use React-Query polling against a session-validated edge function instead.

### Getting changes to the deployed branch
Lovable deploys from `main`. A **stacked PR whose base is a feature branch only reaches `main` if that base merges *after* it** — otherwise the stacked changes land on the (already-merged) feature branch and never reach `main`, and Lovable "can't find" the new files. **Base edge-function/migration PRs on `main`** (or, for a stack, re-target to `main` before merging). After merge, ask Lovable to deploy the named functions / apply the migration from `main`.

---
> Source: [tderi33/spec-compliant-kit](https://github.com/tderi33/spec-compliant-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
