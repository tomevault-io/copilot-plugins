## no-block-pls

> - `src/lib.rs`: Core instrumentation logic (inserts timing guards).

# Repository Guidelines

## Project Structure & Module Organization

- `src/lib.rs`: Core instrumentation logic (inserts timing guards).
- `src/transformer.rs`: AST transforms + unit/snapshot tests.
- `src/guard.rs`: Lightweight runtime guard used in instrumented code.
- `src/main.rs`: CLI (built with `clap`).
- `src/snapshots/*.snap`: `insta` snapshot outputs.
- `test_fixtures/*.rs`: Larger real‑world async samples for tests.

## Build, Test, and Development Commands

- Build: `cargo build` (use `--release` for performance).
- Run CLI preview: `cargo run -- src/my_async.rs`.
- Run CLI in‑place: `cargo run -- --in-place src/` (creates `.bak` backups).
- Tests (unit + snapshot): `cargo test`.
- Snapshot review: `cargo insta review` (accept/deny snapshot updates).
- Format: `cargo fmt --all`.
- Lint: `cargo clippy --all-targets -- -D warnings`.

## Coding Style & Naming Conventions

- Rust edition: 2024. Use idiomatic Rust, 4‑space indent.
- Names: modules/files `snake_case`; types/traits `CamelCase`; functions
  `snake_case`.
- Public API in `lib.rs` should have doc comments and minimal surface area.
- Keep `guard.rs` lean; threshold tweaks belong near its `Duration` check.
- Run `cargo fmt` and `cargo clippy` before pushing.

## Testing Guidelines

- Prefer small, focused unit tests near code (`#[cfg(test)]` modules).
- Use `insta` for transformed‑code assertions; update by running tests then
  `cargo insta review`.
- Test names: start with `test_...` and describe behavior, e.g.,
  `test_async_block_comprehensive`.
- Large inputs live in `test_fixtures/`; keep them stable to avoid noisy
  snapshots.

## Commit & Pull Request Guidelines

- Commits: follow Conventional Commits, e.g.,
  `feat: add tokio::select handling`, `fix: preserve lifetimes in async blocks`.
- PRs must include: clear description, rationale, before/after examples (or CLI
  output), updated tests/snapshots, and any perf notes.
- Do not commit `.bak` files from `--in-place`; add `*.bak` to `.gitignore`
  locally if needed.  

---
> Source: [0xdeafbeef/no-block-pls](https://github.com/0xdeafbeef/no-block-pls) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
