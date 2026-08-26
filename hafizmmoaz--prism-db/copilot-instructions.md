## prism-db

> Guidance for AI coding agents working in this repository.

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this project is

PrismDB is a single-node OLTP **multi-model** database engine written in Rust. Relational SQL tables, JSON-like documents, and ordered key-value pairs live on **one** storage engine, sharing one buffer pool, one WAL, and one transaction manager, so a single transaction can atomically mutate all three models.

Workspace version `0.2.0`, Apache-2.0, 17 crates under `crates/`, 7 standalone client SDKs under `sdks/`.

Binaries: `prismd` (server), `prism-shell` (REPL), `prism-fsck`, `prism-dump` - all defined in `crates/prism-cli/src/bin/`.

Explicitly out of scope for v1: distribution/replication, columnar analytics, and Postgres/Mongo/Redis wire compatibility. Do not add these without an ADR.

## Toolchain

Rust **1.85.0** exactly, pinned in `rust-toolchain.toml` (edition 2024, MSRV 1.85). Components: `clippy`, `rustfmt`.

Changing the pinned toolchain or the MSRV **requires an ADR**. The `sqlparser` dependency is deliberately configured with `default-features = false` because its `recursive-protection` feature pulls a build-dep needing Rust > 1.85 - do not "fix" this by enabling default features.

## Commands

Verify with these before declaring work done. CI sets `RUSTFLAGS: "-D warnings"`, so warnings fail the build.

```sh
cargo fmt --all                                                   # format (CI checks --check)
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-targets --all-features --locked
cargo deny check                                                  # advisories, licenses, sources
```

Two aliases exist in `.cargo/config.toml`:

```sh
cargo lint    # clippy --all-targets --all-features -- -D warnings
cargo ci      # test --all-targets --all-features --locked
```

Narrower loops while iterating:

```sh
cargo check                          # fast type-check
cargo test -p prism-wal              # one crate
cargo test -p prism-server --test durability   # one integration test file
cargo test -- --nocapture            # show stdout
cargo doc --no-deps --open           # API docs
```

CI (`.github/workflows/ci.yml`) runs on push to `main` and all PRs: an `fmt` job, a `test` job matrixed over ubuntu/macos/windows (clippy + test), and a `deny` job. All three OSes must pass - avoid Unix-only path or syscall assumptions.

### SDKs

Each SDK is a **standalone reimplementation of the wire protocol** - no FFI, no shared codegen, no native addon. Changing the protocol means updating `crates/prism-protocol` **and all seven SDKs**. Each is built from its own directory:

| SDK | Commands |
| --- | --- |
| `sdks/c`, `sdks/cpp` | `make`, `make test`, `make example`, `make clean` |
| `sdks/node` | `npm ci`, `npm run build`, `npm test`, `npm run example` (Node 22+ to run tests; published SDK supports Node ≥ 20) |
| `sdks/python` | setuptools build, Python ≥ 3.8; single test file `tests/test_codec.py` |
| `sdks/java` | `mvn` per `pom.xml` |
| `sdks/dotnet` | `dotnet` against `PrismDb.slnx` |
| `sdks/php` | `composer` per `composer.json` |

SDK tests are offline codec round-trips with hardcoded byte layouts, so a wire-format change will surface there. There is no cross-SDK conformance harness.

## Architecture: the layering rule

Crate dependencies flow **downward only**, never sideways between engines, never upward, no cycles. This is normative, not advisory - see `docs/architecture/module-layout.md`.

```
clients:  prism-shell   prism-client   prism-sdk-node   prism-cli
             ↓
server:   prism-server            (query dispatch + network)
             ↓
engines:  prism-sql   prism-doc   prism-kv     ← siblings, never depend on each other
             ↓
access:   prism-index             (B+tree, hash)
             ↓
core:     prism-core              (txn manager, MVCC, locks, ARIES recovery, record store)
             ↓
          prism-wal   prism-buffer
             ↓
          prism-storage           (disk manager; no Prism deps)

prism-protocol sits outside the stack: wire format only, shared by server and clients.
```

Four forbidden patterns:

1. `prism-storage` referencing the WAL.
2. `prism-core` referencing any engine.
3. Any engine depending on another engine.
4. Any crate referencing `prism-server` - except `prism-bench`.

The record store, transaction manager, buffer pool, and WAL are **singletons per process**, shared by all three access methods. That sharing is what makes cross-model transactions atomic; preserve it.

### Crate responsibilities

| Crate | Role |
| --- | --- |
| `prism-storage` | Page-based disk manager, checksums, DB header |
| `prism-wal` | Physiological ARIES write-ahead log, segments, records |
| `prism-buffer` | Clock-sweep buffer pool |
| `prism-core` | Transactions, MVCC visibility, lock manager, recovery, record store |
| `prism-index` | Access methods: B+tree today, extendible hash index still to come |
| `prism-sql` | SQL engine, catalog, type system |
| `prism-doc` | Document engine and value model |
| `prism-kv` | Ordered key-value engine |
| `prism-protocol` | Binary wire protocol codec, frames, messages |
| `prism-server` | Dispatcher, network server, auth, authz, TLS, audit, dump |
| `prism-client` | Rust client |
| `prism-shell` | Interactive REPL |
| `prism-cli` | Binary entry points (`prismd`, `prism-shell`, `prism-fsck`, `prism-dump`) |
| `prism-sdk-node` | napi-rs Node binding |
| `prism-fsck` | Consistency checker |
| `prism-bench` | Benchmark harness binary |
| `prism-testkit` | Fault injection (`FaultyDisk`), crash scenarios, `TempDir`, seeded `Rng` |

## Code conventions

Read `docs/project/engineering-standards.md` for the full set. The rules that matter most in practice:

**Module layout.** One module = one flat file. There are **zero `mod.rs` files** in this repo and no nested module trees - match that. Every `lib.rs` follows the same four-part shape:

```rust
//! `prism-wal` - crate-level doc explaining the component, pointing at docs/components/*.md

pub mod error;      // flat, alphabetical `pub mod` list
pub mod record;
pub mod segment;
pub mod wal;

pub use error::{Result, WalError};        // flat `pub use` re-exports
pub use record::{LogRecord, RecordPayload};
pub use wal::{Config, SyncMode, Wal};

// crate-wide newtypes and constants
```

`lib.rs` holds no business logic. Internally, use module paths (`use crate::error::WalError`); across crates, always use the re-export (`prism_core::CoreError`, never `prism_core::error::CoreError`).

**Errors.** One `thiserror` enum per crate in `error.rs`, plus `pub type Result<T> = std::result::Result<T, XError>;`. Cross-crate errors wrap with `#[from]`. Every variant gets a doc comment and an `#[error("...")]` message in lowercase. Real example from `prism-core`:

```rust
/// An error from the write-ahead log.
#[error("WAL error: {0}")]
Wal(#[from] prism_wal::WalError),

/// A write-write conflict under snapshot isolation: another transaction
/// committed a change to this record after our snapshot began. Retry.
#[error("serialization failure (write-write conflict)")]
SerializationFailure,
```

Do not use `anyhow::Error` in library code - it loses type information. `anyhow` is for binary boundaries (`main`, tests) only. Prefer `?` over `match`; only `match` when you actually inspect the error. `let _ = result` needs a comment saying why.

**Panics.** `Result` for anything recoverable. `panic!` only for genuinely unreachable cases or invariants the compiler can't prove. Recovery code may panic when it cannot proceed - the fsync gate pattern.

**Unsafe.** Prefer safe code even when slower. Any `unsafe` block needs a `// SAFETY: ...` comment directly above explaining soundness, plus a test if the reasoning is non-trivial. "It would be faster" and "the safe version is awkward" are not justifications.

**Naming.** Modules `snake_case` and short; types `CamelCase`; functions `snake_case` verb phrases; constants `SCREAMING_SNAKE_CASE`. Abbreviations only for the established set: `txn`, `wal`, `mvcc`, `lsn`.

**Visibility.** Default private, `pub(crate)` for cross-module, `pub` only for real API surface. Note the codebase declares modules `pub` and relies on re-exports for the intended path.

**Newtypes.** Wrap semantically meaningful integers (`PageId`, `Lsn`, `TxnId`) rather than passing bare `u64`. Existing newtypes are tuple structs with `#[inline] const fn` accessors and hand-written `Display`.

**Concurrency.** `parking_lot::Mutex`/`RwLock` over `std::sync` (faster, no poisoning). Document lock ordering whenever more than one lock is in scope. Never hold a lock across `.await` without deliberate reasoning. Choose atomic orderings deliberately and comment non-obvious ones: `Relaxed` for counters, `Acquire`/`Release` for synchronization, `SeqCst` rarely. Avoid unbounded channels - they hide backpressure.

**Async boundary.** The core (record store, buffer pool, WAL) is **synchronous** with explicit blocking calls. Async lives only in the network and SDK layers. Do not make core APIs async. Sync work over a few microseconds goes in `spawn_blocking`.

**Logging.** `tracing` with structured fields, not formatted strings: `tracing::info!(txn_id = ?txn.id(), "transaction committed")`. `?` for `Debug`, `%` for `Display`. Include identifying context (txn/request/connection id).

**Docs.** Public items need doc comments. Use `# Errors` and `# Examples` sections where relevant. Modules implementing non-trivial algorithms open with a `//!` block summarizing the design and linking the design doc under `docs/components/`. Follow that convention: readers navigate between code and the design corpus constantly.

**Clippy allows.** `#[allow(clippy::...)]` without an explanatory comment is rejected in review.

**Dependencies.** Conservative by policy. Pinned in the **workspace** `Cargo.toml`, never per-crate. Before adding one, consider whether 50–200 lines of our own code would do. New deps must also satisfy `cargo deny check`.

## Testing

The bar is high here by design: "a database that loses or corrupts data is worse than no database."

- **Unit tests** inline in `#[cfg(test)] mod tests` alongside the source (~30 files do this). Cover every error case and boundary condition, not just the happy path.
- **Integration tests** in the crate's `tests/` dir, using only the public API. `crates/prism-server/tests/` is the main suite: `durability.rs`, `cross_model.rs`, `dispatch.rs`, `authz.rs`, `tls.rs`, `multidb.rs`, `audit.rs`, `dump.rs`, `network.rs`. Note there is **no workspace-root `tests/` directory**, despite what some docs say.
- **Property tests** with `proptest` for anything with a non-trivial invariant - currently in `prism-core/src/visibility.rs`, `prism-index/src/btree.rs`, `prism-wal/src/record.rs`, `prism-sql/src/types.rs`. The idiom is model-based: run randomized sequences against a slow obviously-correct oracle (`BTreeMap`, a model store) and assert agreement. When a property fails, proptest writes the shrunk counterexample to a `proptest-regressions/` file; per `docs/operations/testing-strategy.md` those get committed so later runs replay them deterministically. No such directory exists yet - if you generate one, commit it.
- **Fault injection** via `prism-testkit`: `FaultyDisk` injects torn/lost/EIO writes and probes the WAL invariant; `run_scenario` is an in-process crash simulator. Use these for durability and recovery work rather than hand-rolling fixtures.

Tests must be deterministic. If a test uses randomness, seed it (`prism_testkit::Rng`) and log the seed on failure. One assertion of intent per test. Name tests for behavior: `commit_after_crash_recovers_data`, not `test1`.

Coverage is reported but is not a gate. The qualitative bar: every public function has a test, and every branch in recovery, MVCC, and WAL code is covered.

## Commits

Conventional Commits, scope is usually the crate name, first line ≤ 72 chars, body wraps at 72.

```
feat(wal): add group commit window configuration

Allow operators to tune the group commit window via prismd.toml.
Defaults to 1ms.

Closes #42.
```

Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`.

## Things that will trip you up

- **Docs sometimes describe intent, not current reality.** Verify against source. Confirmed gaps: `docs/project/engineering-standards.md` shows a `rustfmt.toml` with `imports_granularity`/`group_imports`/`newline_style` and an EditorConfig, but **neither file exists** - formatting is plain `rustfmt` defaults. It also shows a nested `module/mod.rs` layout and crate-root `#![warn(clippy::pedantic, ...)]` attributes; the codebase uses flat files and has **no crate-root lint attributes** at all. It mentions `crossbeam-channel` and `criterion`, which are not in the workspace dependency list. Follow the code for structure; follow the docs for judgment calls.
- **`benches/` directories do not exist.** Benchmarking goes through the `prism-bench` binary: `prism-bench [kv|sql|doc|xmodel|all] [--ops N] [--durable] ...`. See `docs/operations/benchmarking.md`.
- **`--durable` changes benchmark meaning entirely.** Without it the WAL does not fsync; with it every commit fsyncs (the production setting), which dominates write latency. Don't compare numbers across that flag.
- **Dev profile optimizes crypto crates on purpose.** `scrypt`, `salsa20`, `sha2`, and `pbkdf2` are built at `opt-level = 3` even in dev, because scrypt's work factor is punishing unoptimized. Don't remove these overrides to "simplify," and don't weaken scrypt parameters to speed up tests.
- **A pre-existing architectural decision probably has an ADR.** Ten of them live in `docs/adr/`, covering Rust, page-based storage, physiological WAL/ARIES, MVCC snapshot isolation, the unified record format, the single cross-model WAL, the clock-sweep buffer pool, the binary wire protocol, the napi-rs SDK, and the Volcano executor. Read the relevant one before changing a fundamental; propose a new ADR rather than quietly diverging.
- **Default credentials are `admin`/`admin`.** Fine for local dev and the CI smoke test. Never present a setup as production-ready while these are in place.
- **On-disk and wire formats are specified documents.** `docs/specs/page-format.md`, `record-format.md`, `wal-record-format.md`, `wire-protocol.md`. Changing a byte layout means updating the spec, the codec, and the SDK round-trip tests together.

## Where to look

- `docs/overview/executive-summary.md` - one page on the project
- `docs/architecture/system-architecture.md`, `module-layout.md`, `data-flow.md` - components, layering, request path
- `docs/components/*.md` - per-component design (btree, buffer-pool, mvcc, recovery, wal, sql-engine, ...)
- `docs/specs/*.md` - page, record, WAL record, wire protocol, SDK API, shell
- `docs/operations/*.md` - build-and-dev, testing-strategy, fault-injection, benchmarking, observability, install, releasing
- `docs/project/engineering-standards.md`, `code-review-guide.md` - conventions and what reviewers check

---
> Source: [HafizMMoaz/prism-db](https://github.com/HafizMMoaz/prism-db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
