## sdrive-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands
- Build: `cargo build`
- Run: `cargo run`
- Test: `cargo test`
- Run specific test: `cargo test test_name`
- Check: `cargo check`
- Format: `cargo fmt`
- Lint: `cargo clippy`

## Code Style Guidelines
- Use Rust's Result<T, E> for error handling with anyhow for error context
- Follow standard Rust naming conventions (snake_case for functions/variables)
- Organize code into modules matching the directory structure
- Use async/await for all I/O operations
- Document public functions with /// comments
- Keep error messages user-friendly and informative
- Use structured logging with tracing macros
- Group imports by std, external crates, then local modules
- Prefer strong typing over stringly-typed code
- Use appropriate error propagation with the ? operator

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robbestad) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
