## ceramics-check-in

> Use when editing /supabase/ migrations or Edge Functions, or DB tasks: schema updates, RLS policies, cron autoCheckout, realtime channels, env keys, pg tests, db-lint, schema-drift CI.


# Supabase Rules (SQL · Edge Functions)

## Pathing
* **MUST** place SQL migrations in `supabase/migrations/` and name them `YYYYMMDD_HHMM_<slug>.sql`.
* **SHOULD** keep edge-function source in `supabase/functions/<name>/index.ts`.

## Modelling
* **MUST** prefix every table script with `-- migrate:up` / `-- migrate:down`.
* **MUST** include `created_at TIMESTAMP DEFAULT now()` on all tables.
* **SHOULD** store enums in a dedicated `enums/` schema and reference by type.
* **MUST** expose **camelCase** columns to clients (no snake_case); enable **Row Level Security** and add policies so members can read only their own rows.

## Patterns
* **MUST** run `supabase db lint` in CI before migration merges.
* **SHOULD** keep Realtime channels read-only from clients; writes go through RPC or REST.
* **MUST** schedule `autoCheckout` Edge Function with `*/15 * * * *` (matches Free tier minimum).  
* **MUST** never store secrets in SQL—use environment variables (`SUPABASE_SERVICE_ROLE_KEY`, etc.)

## Testing
* **MUST** run `supabase start` integration tests in CI for migrations.
* **SHOULD** snapshot diff the generated schema (`pg_dump`) to detect drift.
* **SHOULD** unit-test `autoCheckout` logic with mocked session rows.

## Tooling
* **MUST** keep `.env.example` listing `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`.
* **MUST** tag migration PRs with `feat(db)` conventional commit scope.

## Migration Patterns
* **MUST** run `supabase db diff` locally, commit the SQL.
* **MUST** include a matching rollback (`-- migrate:down`).
* **SHOULD** squash multiple WIP migrations before release.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kingwilliam12) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
