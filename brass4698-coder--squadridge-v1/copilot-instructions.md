## supabase

> Supabase MCP rules for SquadRidge-V1. Apply when working with database, auth, edge functions, or storage.


# Supabase Rules for SquadRidge-V1

## Project Info
- Project name: SquadRidge-V1
- Project ref: wrptdvwdcknmhclethjh
- Region: us-west-2
- Stack: TypeScript, Supabase (Postgres + Auth + Edge Functions + Storage)

## Database (DDL)
- ALWAYS use `apply_migration` for any DDL (CREATE TABLE, ALTER TABLE, DROP, etc.)
- NEVER use `execute_sql` for schema changes — only for SELECT queries and data reads
- Name migrations in snake_case describing the change (e.g. `add_squad_members_table`)
- Do NOT hardcode generated UUIDs or auto-increment IDs in migrations
- After any DDL change, run `get_advisors` (both security and performance) to catch issues

## Queries
- Use `execute_sql` for read-only queries and data inspection
- Always use parameterized queries — never interpolate user data directly into SQL strings
- Prefer Supabase client SDK over raw SQL when writing application code

## Auth & RLS
- Every table that stores user data MUST have Row Level Security (RLS) enabled
- Always create RLS policies immediately after creating a table
- Use `auth.uid()` in RLS policies to scope data to the authenticated user
- Never disable RLS on a table without explicit user approval

## Edge Functions
- Use `deploy_edge_function` to deploy; always set `verify_jwt: true` unless the function explicitly implements its own auth
- Edge Functions run on Deno — use `jsr:` and `npm:` imports, not Node.js `require()`
- Store secrets via Supabase dashboard Secrets — never hardcode API keys in function code

## TypeScript Types
- After schema changes, regenerate types with `generate_typescript_types`
- Save generated types to `src/types/supabase.ts`

## Security
- Always keep manual tool call approval ON in Cursor
- Never connect MCP to production data with write access during active development
- Scope MCP to this project ref only: `wrptdvwdcknmhclethjh`

## Logs & Debugging
- Use `get_logs` with service type `postgres`, `auth`, `api`, or `edge-function` to debug issues
- Check `get_advisors` regularly — especially after schema changes

---
> Source: [brass4698-coder/squadridge-v1](https://github.com/brass4698-coder/squadridge-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
