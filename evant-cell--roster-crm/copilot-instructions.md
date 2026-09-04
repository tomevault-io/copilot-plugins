## roster-crm

> Project rules for Roster. Voice rules and self-check protocol from `~/.claude/CLAUDE.md` apply.

# CLAUDE.md (roster-crm)

Project rules for Roster. Voice rules and self-check protocol from `~/.claude/CLAUDE.md` apply.

## What this is

Single-user CRM for Evan to track leads through a simple pipeline (new, contacted, qualified, contracted, lost), log activity, snooze follow-ups, import/export CSV, and send email straight from a lead via Gmail. Deployed on Cloudflare Pages.

## Stack

1. Frontend. Static HTML/CSS/JS in `public/`, no framework, no bundler, no build step.
2. Backend. Cloudflare Pages Functions under `functions/`.
3. Database. Supabase Postgres, accessed only from Functions through PostgREST with the service-role key (`functions/lib/db.js`). The frontend never talks to Supabase directly.
4. Sessions and Google OAuth tokens live in Cloudflare Workers KV.
5. No npm dependencies anywhere. Plain JavaScript, ES modules.

## Auth model

1. Google OAuth 2.0, authorization-code flow with PKCE (`functions/lib/google.js`, `functions/api/auth/*.js`).
2. Scopes: `openid email https://www.googleapis.com/auth/gmail.send`.
3. `access_type=offline` and `prompt=consent` on every auth request so a refresh token always comes back.
4. Server-side email allowlist, `ALLOWED_EMAIL` (comma separated), compared lowercase against the ID token's email claim. Any other Google account is rejected with 403 and its tokens are discarded, never written to KV.
5. Refresh token in KV at `oauth:refresh:<email>`. Access token cached at `oauth:access:<email>` with a short TTL. Each allowed user sends from their own Gmail.
6. Session cookie is HttpOnly, Secure, SameSite=Lax, 30 days, carries only a signed session ID. Session record at `session:<id>` in KV.
7. `functions/_middleware.js` gates everything under `/api/*` except `/api/auth/*`, which stays public so the frontend can check `/api/auth/status` and start the login flow while signed out. An unauthenticated request to a protected route gets a 401 JSON body, not a redirect. Static files are always public, the frontend shows its own login screen when signed out.

## Hard rules

1. No em dashes anywhere (code comments, docs, commit messages, UI copy). Use commas, periods, or parentheses.
2. No emojis anywhere.
3. No semicolons in prose docs.
4. Keep code plain and readable, no clever one-liners over clarity.
5. Database access only from Functions, only through PostgREST with the service-role key. Never expose that key to the frontend.
6. Refresh token never leaves the Worker.
7. Email allowlist enforced server-side from ID token claims, never from frontend input.

## Env vars

Set locally in `.dev.vars` (see `.dev.vars.example`), and as Pages secrets in production.

1. `GOOGLE_CLIENT_ID`
2. `GOOGLE_CLIENT_SECRET`
3. `ALLOWED_EMAIL`
4. `SUPABASE_URL`
5. `SUPABASE_SERVICE_KEY`
6. `SESSION_SECRET`

KV binding: `KV` (set in `wrangler.toml`, real namespace IDs come from `wrangler kv namespace create`).

## Common commands

### Local preview

```
wrangler pages dev .
```

Opens `http://localhost:8788`. Needs `.dev.vars` populated first.

### Deploy

```
git push origin main
```

Cloudflare Pages auto-deploys from the connected GitHub repo.

## File map (high level)

1. `public/` static frontend, served as-is.
2. `functions/api/auth/` OAuth login, callback, logout, status.
3. `functions/api/leads/` lead list/create, single-lead get/patch/delete, activities, snooze.
4. `functions/api/import/` CSV preview and commit.
5. `functions/api/export.js` CSV export of active leads.
6. `functions/api/settings.js` app settings (checklist, stale_days, digest config).
7. `functions/api/email/send.js` sends email via Gmail API on behalf of the signed-in user.
8. `functions/api/digest.js` overdue/due-today lead lists.
9. `functions/lib/` shared backend utilities (db client, CSV parser, dates, Google client, session signing, HTTP helpers, import field guessing).
10. `supabase/schema.sql` full database schema, run once in the Supabase SQL editor.

---
> Source: [evant-cell/roster-crm-](https://github.com/evant-cell/roster-crm-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
