## raisfast

> raisfast — Rust-powered high-performance BaaS and headless CMS. Single binary, zero dependencies, zero GC. Built-in blog, ecommerce, wallet, payment & multi-tenant SaaS. JS / Rhai / Lua / WASM plugin engines for infinite extensibility.

# AGENTS.md

## Project

raisfast — Rust-powered high-performance BaaS and headless CMS. Single binary, zero dependencies, zero GC. Built-in blog, ecommerce, wallet, payment & multi-tenant SaaS. JS / Rhai / Lua / WASM plugin engines for infinite extensibility.

- **Crate name:** `raisfast`
- **Rust edition:** 2024
- **Architecture:** Handler → Service → Model three-layer
- **Plugin engines:** JS (QuickJS) / Rhai / Lua (mlua) / WASM (wasmtime)
- **Databases:** SQLite / PostgreSQL / MySQL (feature-gated)

## Commands

```bash
# Compile (SQLite + JS + Rhai)
SQLX_OFFLINE=false DATABASE_URL="sqlite:./storage/db/raisfast.db?mode=rwc" \
  cargo clippy --tests --no-default-features \
  --features "db-sqlite,plugin-js,plugin-rhai" -- -D warnings

# Test
SQLX_OFFLINE=false DATABASE_URL="sqlite:./storage/db/raisfast.db?mode=rwc" \
  cargo test --no-default-features \
  --features "db-sqlite,plugin-js,plugin-rhai"

# Format check
cargo fmt --check
```

## Architecture

```
Handler → Service → Model (SQL)
                ↘ External: Storage / Cache / Search / EventBus
```

- **src/handlers/** — axum route handlers (thin: extract params, call service, return response)
  - Handler layer is the **only** auth entry point (`ensure_*` calls)
- **src/services/** — business logic layer
  - Service layer does **Policy only** (resource ownership checks), never calls `ensure_*`
- **src/models/** — data structures and DB queries (sqlx + CRUD macros)
  - Model provides `tx_*` variants for transaction participation
- **src/middleware/** — JWT auth, rate limiting
- **src/errors/** — unified `AppError` (thiserror) implementing `IntoResponse`
- **src/config/** — env/config loading
- **src/db/** — connection pool, SQL dialect, schema, write lock
- **src/plugins/** — 4-engine plugin system (JS/Rhai/Lua/WASM)
- **src/content_type/** — dynamic content type system
- **src/worker/** — job queue + cron scheduler (infrastructure, not model layer)

## Key Constraints

- **`unsafe` is banned.** `#![deny(unsafe_code)]` at crate root.
- **No `unwrap()` / `expect()`** in non-test code. Use `?` or explicit error handling.
- **Error handling:** `thiserror` for `AppError` at handler boundaries; `anyhow` for internal service propagation.
- **Database:** SQLite via sqlx. All timestamps as TEXT in ISO 8601.
- **Primary keys:** Snowflake ID (ferroid) with multiplicative inverse cipher + base62 encoding.
- **Auth:** JWT (HS256) with short-lived access tokens + DB-stored refresh tokens.
- **Write lock:** All transactions go through `acquire_write()` (tokio Mutex) to serialize SQLite writes and eliminate `SQLITE_BUSY` tail latency.

## CRUD Macro System

All DB operations use the Where DSL macro system (`raisfast-derive`):

- `crud_insert!`, `crud_update!`, `crud_delete!` — write operations
- `crud_find!`, `crud_find_one!`, `crud_find_all!` — read operations
- `crud_find_page!`, `crud_join_paged!` — pagination with JOINs
- `crud_resolve_id!`, `crud_resolve_ids!` — ID resolution
- `in_transaction!` — transaction wrapper (auto-acquires write lock)

## Style

- `cargo fmt` and `cargo clippy` are authoritative.
- Public items require `///` doc comments.
- Handler → Service → Model layering enforced.

---
> Source: [RaisFast/raisfast](https://github.com/RaisFast/raisfast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
