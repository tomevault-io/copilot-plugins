## claude-statusline

> Status line for Claude Code CLI, implemented in Python and Rust. See README.md for usage, installation, and benchmarks.

# Claude Code Status Line

Status line for Claude Code CLI, implemented in Python and Rust. See README.md for usage, installation, and benchmarks.

## Working on this codebase

- `just check` runs everything: lint (ruff + clippy), format check, toml check, tests (Python 3.14 + 3.9 + Rust)
- `just format` before committing — ruff + cargo fmt
- `just build` for Rust release binary
- Python and Rust implementations must produce identical output for the same input
- Both share the same cache files on disk

## Architecture

- **statusline.py** + **test_statusline.py** — Python implementation (117 tests)
- **rust/src/** — Rust rewrite, modular: input, theme, git, cache, usage, update, format
- **rust/tests/integration.rs** — end-to-end binary tests (11 tests)
- **justfile** — all dev commands

## Conventions

- Tests cover observable behavior (output format, colors, thresholds) — not internals
- Background work (API calls, npm checks) uses `fork()` to never block the status line
- Git dirty check is the main perf bottleneck (~25ms); `CLAUDE_STATUSLINE_SKIP_DIRTY` reads `.git/HEAD` directly instead
- Rust code: `cargo clippy -- -D warnings` must pass, no exceptions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ilyannn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
