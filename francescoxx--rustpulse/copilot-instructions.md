## rustpulse

> Rust + Axum + SQLx API serving public Rust ecosystem signals. This mount is

# RustPulse — API service (appdev / appstage)

Rust + Axum + SQLx API serving public Rust ecosystem signals. This mount is
the **monorepo root** (Cargo workspace + migrations + repo docs + `apps/web`).

## Zerops service facts

- HTTP port: `8080`
- Health endpoint: `/api/health` (used by the prod readiness check)
- Sibling: `db` (PostgreSQL) — wired as `DATABASE_URL` in `zerops.yaml`
- Setups: `dev` (appdev, Ubuntu) and `prod` (appstage, Alpine/musl)

## Layout

- `apps/api` — Axum server (lib + bin)
- `apps/worker` — `migrate` / `seed` / `ingest` CLI
- `crates/rustpulse-core` — types, scoring, security status (unit tested)
- `crates/rustpulse-sources` — `DataSource` providers (fixture-backed, live TODOs)
- `crates/rustpulse-ai-bench` — AI benchmark scaffold + mocked leaderboard
- `migrations/` — SQL schema (embedded via `sqlx::migrate!`)
- `apps/web` — Next.js dashboard (canonical copy of the `webdev` service)

## Dev loop (appdev)

1. Deploy `dev` (runs `cargo fetch`).
2. `cargo build` once, then `./target/debug/rustpulse-worker migrate && ./target/debug/rustpulse-worker seed`.
3. Run the API via the dev server: `./target/debug/rustpulse-api` on port 8080.

`DATABASE_URL` is injected from `zerops.yaml`; never hardcode credentials.
SQLx uses **runtime** queries (no `DATABASE_URL` needed at build time).

## Notes

- All platform ops (deploy, env, logs, scaling) go through the `zcp` MCP tools.
- The prod build targets musl; sqlx has no TLS feature (plaintext private
  network) and reqwest uses rustls — both build cleanly on Alpine.

---
> Source: [FrancescoXX/rustpulse](https://github.com/FrancescoXX/rustpulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
