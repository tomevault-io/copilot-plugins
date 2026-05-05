## beatcheck

> - `cargo build` — development build

# Repository Guidelines

## Build, Test, and Lint Commands

- `cargo build` — development build
- `cargo build --release` — release build (stripped, LTO enabled)
- `cargo run` — run the TUI
- `cargo run -- --refresh` — headless feed refresh
- `cargo run -- --import feeds.opml` — import OPML file
- `cargo test` — run all tests
- `cargo test test_name` — run a single test by name (matches substring)
- `cargo test --package beatcheck -- module::tests::test_name` — run specific test
- `cargo clippy` — lint
- `cargo clippy --all-targets --all-features -- -D warnings` — strict lint (CI uses this)
- `cargo fmt` — format code
- `cargo fmt --all -- --check` — check formatting without changes

## Project Structure

```
src/
├── main.rs          # Entry point, terminal setup, main loop
├── app.rs           # Central app state, async channels
├── config.rs        # TOML config loading
├── error.rs         # AppError enum with thiserror
├── blocklist.rs     # Keyword blocking
├── lib.rs           # Library root
├── ai/              # Claude API summarization
├── db/              # SQLite repository (tokio-rusqlite)
├── feed/            # RSS/Atom parsing, OPML import/export
├── models/          # Data structs (Feed, Article, Summary)
├── services/        # External services (Raindrop, content fetching)
└── tui/             # Ratatui rendering and key handlers
```

## Code Style Guidelines

### Imports
- Group imports: std lib first, then external crates, then local `crate::`
- Use `use` statements at module level; avoid deep nesting
- Prefer `crate::error::Result` for function return types

### Naming
- Modules/files: `snake_case`
- Types/structs: `PascalCase`
- Functions/methods: `snake_case`
- Constants: `SCREAMING_SNAKE_CASE`

### Error Handling
- Use `AppError` enum (defined in `error.rs`) with `#[from]` for conversions
- Return `Result<T>` from functions that can fail
- Use `anyhow::Error` as catch-all variant for ad-hoc errors
- Propagate errors with `?` operator; avoid `.unwrap()` in production code

### Async Pattern
- Use `tokio` for async operations
- Communication between async tasks via `tokio::sync::mpsc` channels
- Main loop polls channels every 100ms for results
- Spawn work with `tokio::spawn`, send results back through channels

### Testing
- Tests use `#[cfg(test)]` modules at the bottom of source files
- Use `#[tokio::test]` for async tests
- Use `tempfile::TempDir` for database tests (auto-cleanup)
- Test helpers (like `test_repo()`) create isolated test fixtures

### Documentation
- Add doc comments (`///`) to public functions
- Use `//` comments for internal explanations
- Keep comments concise; prefer self-documenting code

## Key Dependencies

| Crate | Purpose |
|-------|---------|
| ratatui + crossterm | TUI framework |
| tokio | Async runtime |
| tokio-rusqlite | Async SQLite |
| reqwest | HTTP (rustls-tls only) |
| feed-rs + opml | Feed parsing |
| serde + serde_json + toml | Serialization |
| thiserror + anyhow | Error handling |
| chrono | Date/time |

## CI/CD

CI runs on every push and PR:
1. `cargo fmt --all -- --check` — formatting
2. `cargo clippy --all-targets --all-features -- -D warnings` — lint
3. `cargo test --all-targets --all-features` — tests

All checks must pass before merge.

---
> Source: [leolaporte/beatcheck](https://github.com/leolaporte/beatcheck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
