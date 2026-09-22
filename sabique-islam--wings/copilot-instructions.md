## supabase

> Supabase migrations, RLS, and edge function conventions


# Supabase rules

- Never edit existing migration files — add new timestamped SQL.
- Anon must not SELECT `entries` directly; public shares use `get_shared_entry` RPC.
- `user_id`, `share_token`, `parent_id` protected by trigger — shared editors cannot change them.
- Edge function secrets via `supabase secrets set`, not `VITE_*`.
- `auth-send-email` hook must return HTTP 200 even on failure.
- Grant table privileges to `authenticated` / `service_role` for new tables.

---
> Source: [sabique-islam/wings](https://github.com/sabique-islam/wings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
