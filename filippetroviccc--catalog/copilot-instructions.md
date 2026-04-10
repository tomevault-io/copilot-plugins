## catalog

> - `src/` contains the Rust crate. Key modules map to CLI features: `cli.rs` (arg parsing), `config.rs` (TOML config), `indexer.rs` (filesystem walk), `store.rs` (binary store + JSON export), `search.rs`, `analyze.rs`, `roots.rs`, `output.rs`, `util.rs`.

# Repository Guidelines

## Project Structure & Module Organization
- `src/` contains the Rust crate. Key modules map to CLI features: `cli.rs` (arg parsing), `config.rs` (TOML config), `indexer.rs` (filesystem walk), `store.rs` (binary store + JSON export), `search.rs`, `analyze.rs`, `roots.rs`, `output.rs`, `util.rs`.
- `scope/` holds design and requirements docs (e.g., `scope/cli-spec.md`, `scope/testing.md`, `scope/schema.md`). Treat these as the source of truth for behavior changes.
- `Cargo.toml` and `Cargo.lock` define dependencies. `target/` is build output.

## Build, Test, and Development Commands
- `cargo build` builds the debug binary.
- `cargo build --release` builds an optimized binary.
- `cargo run -- <command>` runs the CLI without installing (example: `cargo run -- index`).
- `cargo install --path .` installs `catalog` for direct use.
- `cargo test` runs all tests. Use `cargo test <name>` for a specific test and `cargo test -- --nocapture` for output.

## Coding Style & Naming Conventions
- Rust edition is 2024. Keep module ownership consistent with the existing layout in `src/`.
- Follow standard Rust formatting (rustfmt defaults). Format before landing non-trivial changes.
- Naming: modules/functions in `snake_case`, types/traits in `CamelCase`, constants in `SCREAMING_SNAKE_CASE`.

## Testing Guidelines
- Tests are currently colocated in modules using `#[cfg(test)]` blocks within `src/*.rs`.
- Prefer small, focused unit tests that mirror behaviors in `scope/testing.md`.
- Add integration tests only if behavior spans multiple modules and cannot be captured by unit tests.

## Commit & Pull Request Guidelines
- Commit messages in history are short, imperative, and unscoped (e.g., “Add README”, “Refactor storage to JSON and update CLI/indexing”). Match that style and avoid trailing periods.
- PRs should include: a concise summary, test results (`cargo test`), and any CLI or config changes called out. If you change behavior, update relevant `scope/*.md` docs.

## Configuration & Reference Notes
- Default paths: config at `~/Library/Application Support/catalog/config.toml`, store at `~/Library/Application Support/catalog/catalog.bin`.
- Env overrides: `CATALOG_CONFIG` and `CATALOG_STORE`.
- See `README.md` for user-facing CLI examples and `CLAUDE.md` for architecture and invariants.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/filippetroviccc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/filippetroviccc)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
