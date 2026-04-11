## reputest

> This file gives AI agents enough context to work effectively on the Reputest codebase.

# Agent guide: Reputest

This file gives AI agents enough context to work effectively on the Reputest codebase.

## Project overview

**Reputest** is a Rust web service that:

- Monitors Twitter/X for **#gmgv** (Gives Me Good Vibes) tweets and builds a directed social graph.
- Processes **@reputest @username following?** mentions: fetches who that user follows and stores in `following` table.
- Exposes HTTP endpoints: Good Vibes dashboard, health check, **web login** (X OAuth 2.0), and an **API playground** (call X API v2 paths with the logged-in user’s token).
- Uses a single **bot** token (from DB) for cronjobs and bot replies; **web sessions** (DB, encrypted) for login and playground.

## Tech stack

- **Rust** (edition 2021), **Axum** 0.7, **Tokio**
- **PostgreSQL** via **sqlx** 0.8 (async, `runtime-tokio-native-tls`)
- **Twitter/X API** v2: OAuth 2.0 PKCE, Bearer token, `api.twitter.com`
- **Security**: AES-256-GCM token encryption (`crypto.rs`), rate limiting (tower_governor), security headers (CSP, X-Frame-Options, etc.)

## Layout

| Path | Purpose |
|------|--------|
| `src/main.rs` | Entrypoint: routes, middleware, app state, server bind |
| `src/lib.rs` | Library root, re-exports |
| `src/config.rs` | Env config: port, BASE_URL/CALLBACK_URI (hardcoded), ALLOWED_USERNAME (hardcoded), TwitterConfig |
| `src/handlers.rs` | HTTP handlers: root, health, reputest, **login**, **login/start**, **oauth callback** (at GET `/reputest`), **playground** GET/POST, logout; `AppState` |
| `src/oauth.rs` | OAuth: Bearer header, refresh token, PKCE helpers, auth URL, code exchange |
| `src/db.rs` | DB: tokens (encrypted), sessions (web login), good vibes, vibe scores |
| `src/crypto.rs` | Encrypt/decrypt tokens (AES-GCM), validate config |
| `src/twitter/` | X API: search, tweets, user lookup, `make_authenticated_request` (uses bot token) |
| `src/cronjob.rs` | GMGV hashtag cron, vibe-request handling, **following-query** handling |
| `sql/database_ddl.sql` | Schema: refresh_tokens, access_tokens, **sessions**, users, good_vibes, **following**, views, indexes |
| `scripts/` | Binaries: `authorize_bot`, `refresh_access_token`, `encrypt_token` |

## Conventions

- **Handlers**: Prefer **concrete return types** (e.g. `axum::response::Response`) when a handler has multiple response shapes (redirect vs HTML), to satisfy `Handler<_, _>` and avoid `impl IntoResponse` branch mismatches.
- **Extractors**: Use **`HeaderMap` + `Bytes`** for handlers that need cookies + body (e.g. playground). Using the full **`Request`** extractor can lead to `Handler` not being satisfied in this Axum version.
- **Security**: Never log tokens; use `html_escape()` for any user-derived HTML; validate paths (e.g. playground: only allow paths starting with `2/` for X API v2).
- **Config**: OAuth callback URI is **hardcoded** as `https://reputest.fly.dev/reputest`; allowed username is **hardcoded** as `julian_le_roux` (see `config.rs`).

## App state and routes

- **State**: `AppState { pool: PgPool, base_url, oauth_client_id, oauth_client_secret }` (all optional except pool). Built in `main.rs` from env (`get_base_url().ok()`, `XAPI_CLIENT_ID`, `XAPI_CLIENT_SECRET`).
- **Routes**: `/` (Good Vibes table), `/following` (following relationships table), `/reputest` (GET: “Reputesting!” or OAuth callback when `?code=&state=` present; POST: “Reputesting!”), `/health`, `/login`, `/login/start`, `/playground` (GET/POST), `/logout`.

## Database

- **Schema**: Apply `sql/database_ddl.sql` (includes `sessions` for web login). For existing DBs, run `sql/migrations/001_add_following.sql`.
- **Tokens**: `access_tokens` and `refresh_tokens` store **encrypted** values; use `crypto::encrypt_token` / `decrypt_token`; require `TOKEN_ENCRYPTION_KEY` (32-byte hex).
- **Sessions**: `sessions` table (id UUID, user_id, username, access_token, refresh_token encrypted, expires_at). Session ID is set in a cookie; validate and load via `db::get_session_by_id`.

## Running and testing

- **Run server**: `cargo run` (or `make` if target exists). Requires `DATABASE_URL`, `TOKEN_ENCRYPTION_KEY`; optional `PORT`, `RUST_LOG`, `XAPI_CLIENT_ID`, `XAPI_CLIENT_SECRET` for web login.
- **Tests**: `cargo test`. Many tests need `DATABASE_URL` and a real DB; they create `AppState` with `pool` and optional OAuth fields.
- **Bot auth**: `cargo run --bin authorize_bot`. Scripts in `scripts/` are separate binaries.

### Cursor / cargo proxy error

When running `cargo` (build, test, fmt, clippy) inside Cursor you may see:

`error: unknown proxy name: 'Cursor-2.x-x86_64'; valid proxy names...`

The **Makefile** (see test/format/clippy targets) works around this by running **`unset ARGV0 && cargo ...`** before invoking cargo. Prefer **`make test`**, **`make format`**, **`make clippy`** when in Cursor, or run `unset ARGV0 && cargo <subcommand>` so cargo uses the real toolchain instead of the Cursor proxy. When validating changes (e.g. before committing), use **`make checkall`** to run tests, format, and clippy in one go.

## Useful references

- **README.md** — User-facing overview, quick start, API table, deployment.
- **docs/BOT_SETUP.md** — Twitter app and OAuth setup.
- **.env.example** — Env vars (no secrets).

When adding routes or handlers, keep the above in mind (state type, extractors, return types, and security).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julerex)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/julerex)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
