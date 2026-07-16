## sso

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Dalang SSO: a self-hosted OAuth 2.0 / OpenID Connect provider (a self-hosted
"Sign in with Google") written in Rust, plus a dashboard to manage OAuth clients
and client SDKs in six languages. Public repo: `github.com/dalang-io/sso`.

## Commands

```bash
# Run the server (dev). First run bootstraps an admin from .env; defaults to SQLite.
cargo run -p sso-server

# Build / release build
cargo build -p sso-server
cargo build --release -p sso-server

# Lint & format (CI enforces both, with -D warnings on clippy)
cargo fmt --all
cargo clippy --workspace --all-targets -- -D warnings

# Tests (whole workspace, or one test by name)
cargo test --workspace
cargo test -p sso-server <test_name>

# JS SDK
cd sdks/js && npm install && npm run build

# Deploy to production (see docs/DEPLOY.md)
./deploy/deploy.sh
```

Run with a scratch DB and PQC signing while iterating:
```bash
DATABASE_URL="sqlite://data/dev.db?mode=rwc" SSO_TOKEN_SIGNING_ALG=ml-dsa-65 \
  SSO_BIND_ADDR=127.0.0.1:8099 cargo run -p sso-server
```

## Architecture

Cargo workspace. The server (`crates/server`, binary name `sso`) is one lean
axum/tokio binary; `sdks/rust` is the Rust client SDK. Other SDKs (`sdks/{js,
go,python,java,php}`) are standalone, not workspace members.

**Three properties make the "100M concurrent users" target tractable — preserve
them when changing code:**

1. **Stateless app tier.** Any node serves any request. The only server-side
   session is the admin dashboard cookie, which is *signed* (`axum-extra`
   `SignedCookieJar`), not stored. Don't introduce server-side session state.
2. **Token verification never hits the DB.** Access/id tokens are self-verifying
   JWTs (`crates/server/src/signing.rs`). Only refresh-token issuance/rotation
   and client lookups touch storage. Keep the hot verification path DB-free.
3. **Runtime-selected storage.** One `sqlx::Any` pool; the driver comes from the
   `DATABASE_URL` scheme (see below). No recompile to switch DBs.

### Request flow (server modules)

- `main.rs` — boot: load `.env` → `config` → connect `db` → bootstrap admin →
  build `signer` → assemble router → serve with graceful shutdown.
- `config.rs` — all env config, with dev-friendly defaults. Every knob is here.
- `state.rs` — `AppState` (cloned into every handler): config, db, signer,
  compiled templates, cookie key. Templates are **embedded** via `include_str!`
  so the binary is self-contained (no runtime template dir).
- `oauth/` — provider endpoints: `authorize` (end-user login/consent + code
  issuance), `enduser` (end-user accounts + session), `token` (all grants +
  PKCE + refresh rotation), `userinfo`, plus discovery + JWKS in `mod.rs`.
  `Claims` and the RSA JWT helpers live in `oauth/mod.rs`.
- `web/` — dashboard: `mod.rs` (admin login/logout, `require_admin` guard,
  routes) and `clients.rs` (client CRUD, origins/redirect-URI editing).

**Multi-tenancy + RBAC.** `tenants` are isolated workspaces that own clients.
Dashboard users (`admins`) have a `role` and a `tenant_id`:
- `super` — global (`tenant_id` NULL); manages tenants + members (`web/admin.rs`,
  super-only pages `/dashboard/tenants` + `/dashboard/members`) and every client.
- `manager` — own tenant; create/delete clients, edit config, manage secrets.
- `developer` — own tenant; add/delete secrets only (no client CRUD/config).
Permission logic lives on `models::Admin` (`can_manage_clients` / `_secrets` /
`_members`, `can_access_tenant`). **Every client handler must call
`client_in_scope` (404s cross-tenant) and re-check the role method** — the UI
hides controls but the server is the gate (verified: developer edit/create/
delete → 403, cross-tenant access → 404). Clients carry `tenant_id`; super's
create form picks the tenant, managers use their own. A "Default" tenant is
ensured at startup (`ensure_default_tenant`) so upgrades and fresh onboards
always have somewhere to put clients.

**Two distinct identities — do not conflate them.** `admins` (members) manage
the dashboard (`web/`, cookie `sso_admin`); `users` are end users who sign in to
relying apps (`oauth/enduser.rs`, cookie `sso_end_user`). `GET /oauth/authorize`
shows the end-user login/registration screen when no `sso_end_user` session
exists, then the consent screen. The authorization code's subject is taken from
the **session**, never from the request body — `POST /oauth/authorize` returns
`access_denied` if unauthenticated, so a browser cannot choose whom it logs in
as. Both sessions are stateless signed cookies (no server-side store).
- `db/mod.rs` — the entire storage surface (`Db`), portable SQL, placeholder
  rewriting.
- `crypto.rs` — Argon2 hashing, token/PKCE helpers, and the **RSA** signing
  `Keys`. `signing.rs` — the `Signer` enum that picks RSA vs ML-DSA.

### Storage portability (non-obvious, easy to break)

The same schema (`crates/server/migrations/0001_init.sql`) runs on SQLite,
Postgres and MySQL/MariaDB because the code holds to a strict portable subset —
see `docs/DATABASE.md`. When touching `db/mod.rs` or the schema:

- IDs are UUID **strings**, timestamps are RFC3339 **TEXT generated in Rust**
  (never `now()`), booleans are `0/1` INTEGER, lists are JSON TEXT.
- Write bind placeholders as `?`; `Db::q()` rewrites them to `$1,$2,…` for
  Postgres. If you hand-write `$1`, it breaks SQLite/MySQL.
- The schema is applied idempotently at boot by splitting on `;` — so **never
  put a `;` inside a SQL comment** (comment lines are stripped, but keep prose
  simple). `CREATE TABLE IF NOT EXISTS` only.
- Oracle/MSSQL are deliberately unsupported by the `Any` pool (no sqlx driver);
  they plug in behind `Db`'s methods later.

### Token signing & PQC

`signing.rs` exposes a `Signer` enum with two backends behind one JWS interface:
`rs256` (default, RSA via `jsonwebtoken`) and `ml-dsa-65` (post-quantum, FIPS
204 via the `fips204` crate, hand-rolled compact JWS with `alg: "ML-DSA-65"`).
Selected by `SSO_TOKEN_SIGNING_ALG`. Both publish a JWKS and are advertised in
discovery. If you add a grant or claim, mint via `state.signer.sign(&Claims)`
and verify via `state.signer.verify(...)` — never call a backend directly.
Details and threat model: `docs/PQC.md`.

## Secrets & git hygiene

`.gitignore` deliberately excludes `.claude/` and `.env` (and `*.db`, key files,
SDK build dirs) — this is a **public** repo. Keep it that way; never commit real
secrets or the JWT private key. `.env.example` is the committed template.
`deploy.sh` never overwrites a remote `.env`.

## Client secrets model

`client_id` is public. A client holds **up to 2 secrets** (`MAX_CLIENT_SECRETS`)
in the `client_secrets` table — separate rows, each with its own `created_at`
(displayed) and a short non-sensitive `hint`. Two slots enable **zero-downtime
rotation**: add a new secret, roll it out, then delete the old one; both
authenticate during the overlap. Each plaintext is shown **exactly once**
(`client_created.html` for the first, `secret_created.html` for rotations);
only Argon2 hashes are stored. The token endpoint (`oauth/token.rs`) accepts a
presented secret if it matches **any** of the client's secrets. Rotating a
secret does not touch the client's origins/redirect-URIs/email allow-list (a
separate form). The legacy `clients.client_secret_hash` column is retained (kept
`''`) for schema compatibility but unused — secrets live only in `client_secrets`.
Redirect URIs are matched exactly; JS origins are the CORS allow-list for
browser/PKCE flows.

## Per-client email allow-list

Each client has `allowed_emails` (JSON TEXT column, same storage pattern as
`js_origins`). `Client::email_allowed` / `models::email_allowed` decide access:
patterns are `@domain` / `*@domain` (whole domain) or `user@domain` (one exact
address), case-insensitive; an **empty list allows everyone** (the dashboard
shows an abuse warning in that state). Enforced at three points — login,
register (rejects *before* creating the account), and, as the authoritative
hard gate, `authorize::decide` right before the code is minted (so a stale
session or bypassed UI still can't get a code). When adding new flow entry
points, re-check `email_allowed` — do not rely on the UI having filtered.

---
> Source: [dalang-io/sso](https://github.com/dalang-io/sso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
