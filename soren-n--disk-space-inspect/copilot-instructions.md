## disk-space-inspect

> - `src/main.rs` bootstraps the `eframe` window and registers the application state in `app.rs`.

# Repository Guidelines

## Project Structure & Module Organization
- `src/main.rs` bootstraps the `eframe` window and registers the application state in `app.rs`.
- `src/app.rs` orchestrates UI state, message handling, and rendering; supporting modules live in `src/fs.rs`, `src/query.rs`, `src/scanner.rs`, `src/tree.rs`, and `src/util.rs`.
- `Cargo.toml` defines dependencies (egui UI stack, background utilities); `Cargo.lock` is checked in to pin versions.
- There is no dedicated assets or tests directory yet—add new modules under `src/` and keep helper files alongside their consumers.

## Build, Test, and Development Commands
- `cargo check` — fast validation of the codebase; run before every commit.
- `cargo fmt` — applies the Rust formatter; required prior to opening a PR.
- `cargo run --release` — launches the desktop UI with optimizations for realistic performance.
- `cargo run` — debug build with quicker rebuild times while iterating on features.

## Coding Style & Naming Conventions
- Follow idiomatic Rust: 4-space indentation, snake_case for modules/functions, CamelCase for types, SCREAMING_SNAKE_CASE for constants.
- Keep UI labels and status strings concise; prefer `format!` with named variables over concatenation.
- Long workflows should be decomposed into helper functions inside the relevant module.

## Testing Guidelines
- Leverage `cargo test` as integration grows; organize tests in `tests/` for integration flows or `mod tests` blocks for unit coverage.
- Mock filesystem interactions by creating temporary directories (`tempfile` crate) when adding tests that touch disk state.
- Name tests with intent, e.g., `test_parse_query_with_size_filter`.

## Commit & Pull Request Guidelines
- Write commits in imperative mood (`Add scanner worker cancellation`), grouping related changes per commit.
- Ensure each PR includes: a clear summary, testing evidence (command output), and links to issues or design docs when applicable.
- Screenshot or screencast UI updates whenever visuals change; attach sample queries that were exercised.

## Architecture Overview
- The UI thread (`app.rs`) receives streamed filesystem updates over `crossbeam-channel` from the background `scanner` thread.
- Queries combine glob filters (`globset`) and size constraints parsed in `query.rs`; `tree.rs` aggregates sizes for directory nodes.
- When extending the scanner, keep work off the UI thread and prefer additional channel messages over shared mutable state.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/soren-n)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/soren-n)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
