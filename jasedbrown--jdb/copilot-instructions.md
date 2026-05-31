## jdb

> - `src/main.rs` bootstraps CLI parsing, logging, and the event loop; `src/options.rs` defines `pid`/`name` launch modes and validation.

# Repository Guidelines

## Project Structure & Module Organization
- `src/main.rs` bootstraps CLI parsing, logging, and the event loop; `src/options.rs` defines `pid`/`name` launch modes and validation.
- `src/debugger.rs` is the core dispatcher; `src/process/*` handles ptrace-driven inferior control; `src/tui/*` renders the terminal UI and routes key events.
- Logs write to `logs/app.log`; keep them local (not ignored by Git). Cargo outputs go to `target/`.

## Build, Run, and Development Commands
- `cargo build` compiles the debugger.
- `cargo run -- name ./path/to/target [args]` launches and debugs a new process; `cargo run -- pid <PID>` attaches to an existing one.
- `cargo fmt` enforces formatting; `cargo clippy -- -D warnings` lints strictly.
- `cargo test` executes the test suite (add tests alongside new features).

## Coding Style & Naming Conventions
- Use Rust 2024 defaults: rustfmt (4-space indent, trailing commas, grouped imports) is canonical.
- Keep modules/files snake_case; types CamelCase; functions and variables snake_case; prefer descriptive channel names.
- Favor `anyhow::Result` + `?` for error flow; avoid `unwrap` in user-facing paths. Emit diagnostics with `tracing`/`log` macros at appropriate levels.

## Testing Guidelines
- Add unit tests with `#[cfg(test)]` near the code they cover; place integration tests in `tests/` for multi-module behavior or process/TUI flows.
- Cover new branches and error handling; prefer deterministic fixtures over real processes when possible. Run `cargo test` before publishing.

## Commit & Pull Request Guidelines
- Follow the existing concise style: short, scope-prefixed, imperative summaries (e.g., `registers: fill out gp set`, `build: set toolchain version`). One line ~50 chars.
- PRs should include: brief summary and rationale, commands run, screenshots/recordings for TUI changes, linked issues, and noted risks or manual verification steps. Ensure `logs/` and `target/` stay out of commits.

## Security & Configuration Tips
- ptrace-based debugging requires appropriate permissions (Linux/`procfs`). Only attach to processes you are allowed to inspect.
- Logs may contain paths and signals; scrub or delete before sharing. Keep any history/config files free of secrets.

---
> Source: [jasedbrown/jdb](https://github.com/jasedbrown/jdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
