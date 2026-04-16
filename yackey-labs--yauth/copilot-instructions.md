## yauth

> `yauth` is a modular, plugin-based authentication library for Rust (Axum) with TypeScript client + SolidJS UI packages. It provides email/password, passkey (WebAuthn), MFA (TOTP + backup codes), OAuth, bearer tokens (JWT), API keys, and admin endpoints — all behind feature flags.

# CLAUDE.md — yauth

## What This Is

`yauth` is a modular, plugin-based authentication library for Rust (Axum) with TypeScript client + SolidJS UI packages. It provides email/password, passkey (WebAuthn), MFA (TOTP + backup codes), OAuth, bearer tokens (JWT), API keys, and admin endpoints — all behind feature flags.

**Repo:** `github.com/yackey-labs/yauth`

## Workspace Structure

### Rust Crates (`crates/`)

| Crate | Purpose |
|---|---|
| `yauth` | Main library — plugins, middleware, builder, auth logic, backends, repository traits |
| `yauth-entity` | Domain types (User, Session, Password, etc.) — ORM-agnostic, no migration dependency |
| `yauth-migration` | Schema types, DDL generation, diff engine, migration file gen — **zero ORM deps**, build-time-only dep of `cargo-yauth` (NOT a dependency of `yauth` itself) |
| `cargo-yauth` | CLI binary — `cargo yauth init/add-plugin/remove-plugin/status/generate` |

Key internal modules in `yauth`:
- `backends/diesel_pg/` — PostgreSQL backend (`DieselPgBackend`)
- `backends/diesel_mysql/` — MySQL/MariaDB backend (`DieselMysqlBackend`)
- `backends/diesel_libsql/` — SQLite/Turso backend (`DieselLibsqlBackend`)
- `backends/diesel_sqlite/` — Vanilla SQLite backend (`DieselSqliteBackend`)
- `backends/sqlx_pg/` — PostgreSQL backend via sqlx (`SqlxPgBackend`)
- `backends/sqlx_mysql/` — MySQL backend via sqlx (`SqlxMysqlBackend`)
- `backends/sqlx_sqlite/` — SQLite backend via sqlx (`SqlxSqliteBackend`)
- `backends/memory/` — In-memory backend (`InMemoryBackend`)
- `backends/redis/` — Redis caching decorators
- `repo/` — `DatabaseBackend` trait (provides `repositories()`), repository traits, `Repositories` struct, `RepoError`
- `domain/` — ORM-agnostic domain types (always compiled, no backend deps)

Key modules in `yauth-migration`:
- `types` — `TableDef`, `ColumnDef`, `ColumnType`, `ForeignKey`, `Dialect`
- `core` — Core table definitions (users, sessions, audit_log)
- `plugin_schemas` — Schema definitions for each plugin
- `collector` — Schema collection + topological sort by FK deps
- `postgres/sqlite/mysql` — Dialect-specific DDL generators
- `diff` — Schema diff engine (CREATE TABLE, DROP TABLE, ALTER TABLE)
- `generate` — Migration file generators (diesel up.sql/down.sql, sqlx numbered .sql)
- `config` — `yauth.toml` config file support
- `tracking` — Schema hash computation

### TypeScript Packages (`packages/`)

| Package | Purpose |
|---|---|
| `@yackey-labs/yauth-shared` | Shared types (`AuthUser`, `AuthSession`, AAGUID map) |
| `@yackey-labs/yauth-client` | HTTP client for all auth endpoints |
| `@yackey-labs/yauth-ui-vue` | Vue 3 components + composables (LoginForm, useAuth, etc.) |
| `@yackey-labs/yauth-ui-solidjs` | SolidJS components (LoginForm, RegisterForm, etc.) |

## Key Commands

```bash
# Rust
cargo test --features full,all-backends          # Run all unit tests
cargo fmt --check                                 # Format check
cargo clippy --features full,all-backends -- -D warnings  # Lint

# TypeScript
bun install                          # Install dependencies
bun run build                        # Build all TS packages
bun run lint                         # Biome lint check
bun run lint:fix                     # Biome lint + fix
bun run typecheck                    # TypeScript type check
bun validate                         # lint:fix + typecheck + build
bun validate:ci                      # lint + typecheck + build + generate:check (strict)

# Client generation
bun generate                         # Regenerate TS client from Rust types + routes
bun generate:check                   # Fail if generated client is out of date (CI)

# Integration / Pentest (all parallel-safe — no --test-threads=1 needed)
docker compose up -d                 # Start PostgreSQL + MySQL + Redis + Mailpit
cargo test --features full,all-backends --test pentest                      # OWASP pentest (memory + diesel_pg + diesel_mysql)
cargo test --features full,all-backends --test diesel_integration           # Diesel PG integration tests
cargo test --features full,all-backends --test diesel_mysql_integration     # Diesel MySQL integration tests

# Conformance tests (cross-backend repository trait verification)
# All tests share a single tokio runtime via OnceLock, so connection pools survive across tests.
DATABASE_URL=postgres://yauth:yauth@127.0.0.1:5433/yauth_test \
MYSQL_DATABASE_URL=mysql://yauth:yauth@127.0.0.1:3307/yauth_test \
  cargo test --features full,all-backends --test repo_conformance

# Schema generation CLI (generates migration files for your ORM — no runtime migrations)
cargo yauth init --orm diesel --dialect postgres --plugins email-password,passkey
cargo yauth add-plugin mfa
cargo yauth remove-plugin passkey
cargo yauth status
cargo yauth generate --check -f yauth.toml
```

### Conformance Test Suite

`tests/repo_conformance.rs` contains 65 tests that verify every repository trait method behaves identically across all 8 backends (memory, diesel_pg, diesel_mysql, diesel_sqlite, diesel_libsql, sqlx_pg, sqlx_mysql, sqlx_sqlite). Tests are parameterized via `test_backends()` — backends are skipped if their database URL env var is unset.

The suite covers three categories:

1. **Method coverage** — every method on every repository trait has at least one test that creates data, reads it back, and asserts specific return values (not just `is_ok()`).
2. **Behavioral contracts** — semantic invariants the type system can't enforce: expired tokens return `None`, used tokens return `None`, `user.delete()` cascades to all related entities, rate limiting is fail-open on error.
3. **Type edge cases** — UUID round-trip (CHAR(36) ↔ Uuid), NULL vs empty string, large text (no silent truncation), unicode/emoji, datetime precision, JSON structural equality, case-insensitive email.

**When adding a new backend:** implement `DatabaseBackend` + all repository traits, add the backend to `test_backends()`, and run the suite. If all 65 tests pass, the backend is correct.

**Schema setup in tests:** Conformance and integration tests set up the database schema using raw SQL via test helper functions (not `backend.migrate()`). Each test backend has a setup function that executes DDL directly against the database connection. This keeps the test infrastructure independent of the migration system.

**Shared runtime pattern:** All conformance tests use `#[test]` (not `#[tokio::test]`) with a shared `OnceLock<Runtime>`. This is critical — each `#[tokio::test]` creates its own tokio runtime, and connection pools are bound to the runtime that created them. When one test's runtime shuts down, shared pool connections die for other tests. The shared runtime ensures all pools and connections live on one runtime that outlives all tests. This enables safe parallel execution (`--test-threads=N`).

## Feature Flags

### Backend Features

| Feature | What It Enables | Default |
|---|---|---|
| `diesel-pg-backend` | PostgreSQL backend via diesel-async + deadpool | Yes |
| `diesel-libsql-backend` | SQLite/Turso backend via `diesel-libsql` crate | No |
| `diesel-mysql-backend` | MySQL/MariaDB backend via diesel-async + deadpool | No |
| `diesel-sqlite-backend` | Vanilla SQLite backend via libsqlite3-sys | No |
| `sqlx-pg-backend` | PostgreSQL backend via sqlx | No |
| `sqlx-mysql-backend` | MySQL backend via sqlx | No |
| `sqlx-sqlite-backend` | SQLite backend via sqlx | No |
| `memory-backend` | Fully in-memory backend (no database required) | No |

### Plugin Features

| Feature | What It Enables | Default |
|---|---|---|
| `email-password` | Registration, login, verification, forgot/reset/change password | Yes |
| `passkey` | WebAuthn registration + login | No |
| `mfa` | TOTP setup/verify + backup codes | No |
| `oauth` | OAuth2 provider linking | No |
| `bearer` | JWT access/refresh tokens | No |
| `api-key` | API key generation + validation | No |
| `admin` | User management, ban/unban, impersonation | No |
| `telemetry` | Native OpenTelemetry SDK instrumentation (spans, span events, context propagation) | No |
| `openapi` | utoipa OpenAPI spec generation (for client codegen) | No |
| `redis` | Redis caching decorator — wraps repository traits for sub-ms session/rate-limit lookups | No |
| `full` | All auth plugins only — does NOT include any backend (pick one separately) | No |
| `all-backends` | Every backend + redis (CI-only, for conformance testing — excludes diesel-libsql due to symbol conflicts) | No |

Real apps use `full` + one backend (e.g., `features = ["full", "diesel-pg-backend"]`). CI uses `full,all-backends`.

Feature flags gate code across all Rust crates in the workspace.

## Architecture

### Plugin System

Plugins implement the `YAuthPlugin` trait:
- `public_routes()` — unauthenticated routes (login, register, etc.)
- `protected_routes()` — routes behind auth middleware (change password, passkey management, etc.)
- `on_event()` — react to auth events (MFA intercepts login, etc.)

### Database Backends

yauth uses a `DatabaseBackend` trait with multiple implementations. The trait provides `repositories()` — there is no `migrate()` method. Schema creation is handled externally by your ORM's migration tool (`diesel migration run`, `sqlx migrate run`, etc.) using migration files generated by `cargo yauth generate`.

All backends accept pre-configured pools or connections — they do not create their own connections from URLs.

| Backend | Type | Constructor | Use case |
|---|---|---|---|
| `DieselPgBackend` | `backends::diesel_pg` | `from_pool(pool)` / `from_pool_with_schema(pool, "auth")` | Production PostgreSQL (default) |
| `DieselMysqlBackend` | `backends::diesel_mysql` | `from_pool(pool)` | MySQL 8.0+ or MariaDB 10.6+ |
| `DieselLibsqlBackend` | `backends::diesel_libsql` | `from_pool(pool)` | Local SQLite files or remote Turso databases |
| `DieselSqliteBackend` | `backends::diesel_sqlite` | `from_pool(pool)` | Vanilla SQLite via libsqlite3-sys |
| `SqlxPgBackend` | `backends::sqlx_pg` | `from_pool(pool)` | PostgreSQL via sqlx |
| `SqlxMysqlBackend` | `backends::sqlx_mysql` | `from_pool(pool)` | MySQL via sqlx |
| `SqlxSqliteBackend` | `backends::sqlx_sqlite` | `from_pool(pool)` | SQLite via sqlx |
| `SeaOrmPgBackend` | `backends::seaorm_pg` | `from_connection(db)` | PostgreSQL via SeaORM 2.0 |
| `SeaOrmMysqlBackend` | `backends::seaorm_mysql` | `from_connection(db)` | MySQL via SeaORM 2.0 |
| `SeaOrmSqliteBackend` | `backends::seaorm_sqlite` | `from_connection(db)` | SQLite via SeaORM 2.0 |
| `InMemoryBackend` | `backends::memory` | `new()` | Tests, prototyping, CI — no database required |

Redis (`with_redis()`) is a **caching decorator** that wraps repository traits for sub-millisecond session/rate-limit lookups. The database remains the source of truth. Redis is not a separate store backend.

### Builder Pattern

`build()` is **async** and returns `Result<YAuth, RepoError>`. Schema must already exist — use `cargo yauth generate` to produce migration files, then apply them with your ORM's CLI before starting the app.

```rust
use yauth::backends::diesel_pg::DieselPgBackend;

// Your app creates and owns the pool. yauth borrows it.
let pool = /* your diesel-async deadpool */;
let backend = DieselPgBackend::from_pool(pool);

let yauth = YAuthBuilder::new(backend, config)
    .with_email_password(ep_config)
    .with_passkey(pk_config)
    .with_bearer(bearer_config)
    .with_mfa(mfa_config)
    .build()
    .await?;

let router = yauth.router();        // Axum Router<YAuthState>
let state = yauth.state().clone();   // For app-level state sharing
```

### Tri-Mode Auth Middleware

The auth middleware checks credentials in order:
1. **Session cookie** — `CookieJar` → `validate_session()`
2. **Bearer token** — `Authorization: Bearer <jwt>` → JWT validation (feature-gated)
3. **API key** — `X-Api-Key: <key>` → key hash lookup (feature-gated)

Authenticated user is injected as `Extension<AuthUser>` on the request.

### Core Routes (always available)

- `GET /session` — returns authenticated user info
- `POST /logout` — invalidates session cookie

## Generated TypeScript Client

This project uses `utoipa` (OpenAPI 3.1 spec generation) + `orval` (TypeScript client generation from OpenAPI) to auto-generate `@yackey-labs/yauth-client`.

**Pipeline:** Rust types derive `ToSchema` → `routes_meta.rs` builds OpenAPI spec programmatically → `openapi.json` → orval generates `packages/client/src/generated.ts` → `packages/client/src/index.ts` wraps generated functions into backward-compatible `createYAuthClient()` factory.

**Key files:**
- `crates/yauth/src/routes_meta.rs` — OpenAPI spec builder (paths + schemas per feature flag)
- `openapi.json` — generated OpenAPI 3.1 spec (committed)
- `orval.config.ts` — orval configuration
- `packages/client/src/mutator.ts` — custom fetch wrapper (credentials, error handling, bearer tokens)
- `packages/client/src/generated.ts` — orval-generated functions (committed, do not edit manually)
- `packages/client/src/index.ts` — `createYAuthClient()` wrapper providing grouped API

**When modifying any API endpoint or request/response type:**
1. Update the route metadata in `crates/yauth/src/routes_meta.rs`
2. Ensure request/response types have `#[derive(TS)] #[cfg_attr(feature = "openapi", derive(utoipa::ToSchema))] #[ts(export)]`
3. Run `bun generate` to regenerate `openapi.json` and the TypeScript client
4. Commit the regenerated `openapi.json` and `packages/client/src/generated.ts` alongside Rust changes

**CI check:** `bun generate:check` (part of `bun validate:ci`) fails if the OpenAPI spec is out of date.

## Versioning

Releases are fully automated via [knope](https://knope.tech) + GitHub CI. The only manual step is writing conventional commits — everything else happens automatically.

### Happy path
1. Write conventional commits (`feat:`, `fix:`, `feat!:`, etc.)
2. PR is merged to `main`
3. CI runs `.github/workflows/release.yml` which calls `knope release`
4. knope bumps versions in `Cargo.toml`, `Cargo.lock`, all `package.json` files, and writes `CHANGELOG.md`
5. knope commits `chore: prepare release N`, publishes to crates.io + npm, pushes the commit + tag, creates a GitHub release
6. The `chore: prepare release` commit triggers CI again but is skipped by the `if: !startsWith(...)` guard

### Rules
- **NEVER manually edit version numbers** — knope manages `Cargo.toml`, `Cargo.lock`, and all `package.json` files
- **NEVER run `knope release` locally** — use `knope release --dry-run` to preview
- All Rust crates and npm packages share a **single unified version** via `knope.toml`
- `feat:` → minor bump, `fix:` → patch bump, `feat!:` / `fix!:` / `BREAKING CHANGE:` → major bump

### Partial release recovery
If CI publishes to crates.io/npm but fails before pushing the version commit and tag (e.g., "crate already exists" on a re-run):
1. Manually bump all version files to N
2. Commit as `chore: prepare release N`
3. Open a PR, merge it
4. Push the tag: `git tag vN <squash-sha> && git push origin vN`
5. Open a `fix:` PR to trigger the next version

## Conventions

- **Always use Bun** — `bun install`, `bun run`, etc. Never use npm or yarn for running scripts
- **npm publish uses npm CLI** — the release workflow uses `npm publish --provenance` (not `bun publish`) because only the npm CLI supports OIDC trusted publishing; no NPM_TOKEN is required
- **Conventional commits** for all commit messages — this directly drives automated versioning
- **Biome** for TypeScript linting/formatting (not ESLint)
- **`cargo fmt` + `cargo clippy`** for Rust
- **UUIDv7 for all IDs** — use `Uuid::now_v7()` (not v4). Gives time-sortable IDs with better B-tree locality. DB defaults still use `gen_random_uuid()` (v4) as a fallback since PG <18 has no native v7; the Rust side is authoritative.
- **`yauth_` table prefix** on all database tables
- **Configurable PG schema** — use `DieselPgBackend::from_pool_with_schema(pool, "auth")` to isolate yauth tables in a separate PostgreSQL schema (default `"public"`).
- **Timing-safe patterns** — dummy password hash on failed lookups to prevent timing attacks
- **HIBP k-anonymity** — password breach checking via HaveIBeenPwned API (configurable)
- **Rate limiting** — per-operation rate limits (login, register, forgot-password, etc.)
- **Event system** — `AuthEvent` enum emitted on all auth operations for plugin interception

## Telemetry

Uses native OpenTelemetry SDK (no `tracing` crate). Key points:
- `opentelemetry` + `opentelemetry_sdk` + `opentelemetry-otlp` for Rust spans (direct SDK, no tracing bridge)
- Errors/warnings in request handlers are OTel **span events** (not disconnected logs) — visible in Honeycomb trace waterfall
- `crate::otel` helper module provides `record_error()`, `add_event()`, `set_attribute()`, `with_span()` — all compile to no-ops when `telemetry` feature is disabled
- `telemetry::init()` registers `TraceContextPropagator` for W3C traceparent/tracestate propagation
- `telemetry::layer::trace_middleware` creates native OTel server spans with context attached via `cx.attach()` and stored in request extensions
- Operational logging uses `log` crate (not `tracing`) — library does NOT init a log subscriber
- `@opentelemetry/sdk-trace-web` for frontend
- CORS must include `traceparent` and `tracestate` headers
- Health checks excluded from tracing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yackey-labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
