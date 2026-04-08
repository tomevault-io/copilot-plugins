## requisition-workflow-system

> > Full details in AI_CONTEXT.md at the repo root.

# Gemini Context — Requisition Workflow System

> Full details in AI_CONTEXT.md at the repo root.

## Supabase Database Connection

- **Project Ref**: `winfoubqhkrigtgjwrpm`
- **Pooler Host**: `aws-1-eu-west-1.pooler.supabase.com` (NOT `aws-0`)
- **Port**: `5432`
- **User**: `postgres.winfoubqhkrigtgjwrpm`
- **Database**: `postgres`
- **Password**: Ask the user. Never hardcode.
- **psql**: Not available. Use Node.js `pg` client with `ssl: { rejectUnauthorized: false }`.

## Stack

- React + Vite frontend (`client/`), Admin panel (`admin/`)
- Supabase backend: Postgres + Auth + Edge Functions + RLS
- Multi-tenancy: pool model, `org_id` on every table, RLS via JWT claims

## Common Mistakes to Avoid

- `platform_admins` column is `user_id`, NOT `auth_user_id`
- `organization_members` uses `organization_id`, NOT `org_id`
- Git push stderr on Windows is normal — look for `main -> main` to confirm success
- Don't use `supabase db push` — deploy SQL via Node.js `pg` client

## RLS & JWT

All 72 RLS policies use JWT claim helpers (no DB lookups):
- `jwt_org_id()`, `jwt_org_role()`, `jwt_workflow_role()`, `jwt_is_platform_admin()`
- Injected by `custom_access_token_hook()` at token mint time

## Edge Function Deployment

```bash
npx supabase functions deploy <function-name> --project-ref winfoubqhkrigtgjwrpm
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ronnie1560)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Ronnie1560)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
