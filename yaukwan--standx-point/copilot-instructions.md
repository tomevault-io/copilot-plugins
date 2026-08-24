## standx-point

> This file is for coding agents operating in this repository.

# AGENTS.md - StandX Point Workspace Guide

This file is for coding agents operating in this repository.
It documents the real build/test/lint commands and the coding conventions observed in code.

## Scope and Structure

- Rust workspace root: `Cargo.toml` with members:
  - `crates/standx-point-adapter`
  - `crates/standx-point-mm-strategy`
  - `examples/json-persistence-demo`
- Main architecture split:
  - `standx-point-adapter`: StandX protocol/auth/http/ws integration
  - `standx-point-mm-strategy`: strategy orchestration, runtime, TUI/CLI lifecycle

## Toolchain Targets

- `rust-toolchain.toml`: pins the toolchain channel and preinstalls cross-compile targets.

## Rule Precedence (Fractal Context)

- Follow the nearest `AGENTS.md` to the file you edit.
- Root `AGENTS.md` gives workspace defaults.
- Nested `AGENTS.md` files override root guidance for their subtree.
- Many Rust files use a Fractal header block (`[INPUT]/[OUTPUT]/[POS]/[UPDATE]`); preserve it.

## Cursor / Copilot Rules Status

- `.cursor/rules/`: not found
- `.cursorrules`: not found
- `.github/copilot-instructions.md`: not found
- Therefore, repository guidance is from `AGENTS.md` files plus code/config patterns.

## Build / Lint / Test Commands

Run from workspace root unless noted.

### Build

- Build whole workspace:
  - `cargo build --workspace`
- Release build:
  - `cargo build --workspace --release`
- Build one package:
  - `cargo build -p standx-point-adapter`
  - `cargo build -p standx-point-mm-strategy`

### Format and Lint

- Format check:
  - `cargo fmt --all -- --check`
- Format fix:
  - `cargo fmt --all`
- Clippy (workspace, fail on warnings):
  - `cargo clippy --workspace --all-targets --all-features -- -D warnings`
- Clippy (single package):
  - `cargo clippy -p standx-point-mm-strategy --all-targets -- -D warnings`

### Test (Workspace and Package)

- Run all tests in workspace:
  - `cargo test --workspace`
- Run tests for one package:
  - `cargo test -p standx-point-adapter`
  - `cargo test -p standx-point-mm-strategy`
- Run integration test target file:
  - `cargo test -p standx-point-mm-strategy --test integration_test`
  - `cargo test -p standx-point-adapter --test http_tests`

### Single Test Invocation (Important)

- By test name substring in a package:
  - `cargo test -p standx-point-mm-strategy test_full_strategy_lifecycle`
- Exact test name (recommended for stability):
  - `cargo test -p standx-point-mm-strategy test_full_strategy_lifecycle -- --exact --nocapture`
- Exact test inside an integration test file:
  - `cargo test -p standx-point-adapter --test http_tests test_http_user_endpoints_send_bearer_jwt -- --exact --nocapture`
- Run one unit test in a module file by function name:
  - `cargo test -p standx-point-mm-strategy risk_safe_when_no_triggers -- --exact`

### Binary Run Commands

- CLI mode with config:
  - `cargo run -p standx-point-mm-strategy -- --config crates/standx-point-mm-strategy/examples/single_task.yaml`
- Dry run config validation:
  - `cargo run -p standx-point-mm-strategy -- --config crates/standx-point-mm-strategy/examples/single_task.yaml --dry-run`
- TUI mode:
  - `cargo run -p standx-point-mm-strategy -- --tui`

## Code Style Guidelines

These are extracted from current code and local AGENTS docs.

### Imports and Module Layout

- Keep import blocks grouped with blank lines:
  1) `std` imports
  2) third-party crates
  3) `crate::` / `super::` local modules
- Keep public module wiring in `mod.rs` / `lib.rs` minimal and explicit.
- Re-export only intentional public API surface.

### Formatting

- Use `cargo fmt` defaults; do not hand-format around rustfmt.
- Prefer line breaks and trailing commas that rustfmt keeps stable.
- Keep function bodies small where practical; extract helpers instead of nesting deeply.

### Types and Precision

- Use `rust_decimal::Decimal` for price/qty/notional/bps values.
- Do not introduce floating-point (`f32`/`f64`) for trading math.
- Use strong domain enums (`Side`, `OrderType`, `RiskState`, etc.) over stringly logic.
- Parse/convert inputs explicitly; return structured errors on invalid values.

### Naming Conventions

- Rust defaults:
  - `snake_case`: functions/variables/modules
  - `PascalCase`: structs/enums/traits
  - `SCREAMING_SNAKE_CASE`: constants
- Keep names domain-specific (`order_tracker`, `shutdown_token`, `price_jump_threshold_bps`).

### Error Handling

- In adapter/library layers, prefer typed errors (`StandxError`) and crate `Result<T>` aliases.
- In app/binary orchestration, use `anyhow::Result` with `Context` for call-site detail.
- Propagate errors with `?`; avoid `unwrap`/`expect` in non-test paths.
- Do not hide failures; include enough context for operational logs.

### Async, Concurrency, and Shutdown

- Runtime is Tokio.
- Use `tokio_util::sync::CancellationToken` for coordinated shutdown.
- Use `tokio::sync::watch` for market data fan-out snapshots.
- Use `Arc<tokio::sync::Mutex<T>>` for shared mutable state only when ownership redesign is not cleaner.
- Keep task lifecycle boundaries clear:
  - startup/auth/init
  - run loop
  - shutdown/cleanup

### Logging and Observability

- Use `tracing` (`info!`, `warn!`, `debug!`, `error!`) with structured fields.
- Prefer concise, machine-searchable messages; attach key ids (`task_id`, `symbol`).
- Respect current logging setup in `main.rs` (file logging and runtime mode behavior).

### Testing Conventions

- Unit tests: colocated with modules under `#[cfg(test)]`.
- Integration tests: `crates/*/tests/*.rs`.
- HTTP/network-facing tests: use `wiremock`; avoid live external calls.
- Async tests: `#[tokio::test]`.
- Prefer deterministic assertions over timing-fragile checks.

### Architectural Boundaries

- Keep StandX protocol/auth/signature/http/ws logic in `standx-point-adapter`.
- Keep strategy, orchestration, lifecycle, and UI concerns in `standx-point-mm-strategy`.
- Do not leak business strategy logic into adapter data types.

---
> Source: [yaukwan/standx-point](https://github.com/yaukwan/standx-point) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
