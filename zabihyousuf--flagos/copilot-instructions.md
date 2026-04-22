## flagos

> Use Supabase MCP to apply database migrations when available


# Supabase migrations

This project uses Supabase. Migrations live in `supabase/migrations/`.

## Applying migrations

When you need to apply a migration to the linked Supabase project:

1. **Prefer the Supabase MCP server** (if enabled in Cursor settings):
   - Use the **apply_migration** tool with the migration file path, e.g. `supabase/migrations/20250212000000_add_first_down_to_field_settings.sql`, or
   - Use the **execute_sql** tool with the contents of the migration file.

2. **Otherwise** use the Supabase CLI from the project root:
   - `npx supabase db push` (requires the project to be linked: `npx supabase link --project-ref <ref>`).

The project ref is in `.mcp.json` under the Supabase MCP server URL (`project_ref=...`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zabihyousuf) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
