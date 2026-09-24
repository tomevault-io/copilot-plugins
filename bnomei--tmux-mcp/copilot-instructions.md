## tmux-mcp

> - `src/` holds the Rust crate. Key files: `main.rs` (CLI entry), `lib.rs` (library exports), `server.rs` (MCP server), `tmux.rs` (tmux process wrapper), `security.rs` (policy checks/config schema), `commands.rs`, `types.rs`, `errors.rs`, and `test_support.rs` (tmux/ssh stubs for tests).

# Repository Guidelines

## Project Structure & Module Organization
- `src/` holds the Rust crate. Key files: `main.rs` (CLI entry), `lib.rs` (library exports), `server.rs` (MCP server), `tmux.rs` (tmux process wrapper), `security.rs` (policy checks/config schema), `commands.rs`, `types.rs`, `errors.rs`, and `test_support.rs` (tmux/ssh stubs for tests).
- `tests/` contains Rust tests: `cli.rs` for CLI behavior and `integration.rs` for tmux-backed workflows.
- `scripts/` holds release packaging helpers (`build-release`, `package-release`, `resolve-version`).
- `npm/tmux-mcp-rs/`: npm wrapper package (`@bnomei/tmux-mcp-rs`) that downloads the matching GitHub Release binary at first run.
- `Dockerfile`: multi-arch image that fetches musl Linux release assets and runs on Alpine with `tmux`.
- `packaging/README.md` describes npm/Docker/Homebrew packaging surfaces.
- `docs/RELEASE.md` is the release guide.
- Build outputs live in `target/` and `dist/` (both generated).

## Build, Test, and Development Commands
- `cargo build --release` builds the `tmux-mcp-rs` binary in `target/release/`.
- `cargo test --lib` runs unit tests without tmux.
- `cargo test --test cli` runs CLI tests without tmux.
- `TMUX_MCP_INTEGRATION=1 cargo test --test integration` runs tmux integration tests (requires tmux on PATH).
- Release packaging: `TARGET=... VERSION=... scripts/build-release.sh` then `scripts/package-release.sh`.

## Coding Style & Naming Conventions
- Rust 2021 edition; prefer rustfmt defaults (`cargo fmt`) when formatting.
- Use standard Rust naming: `snake_case` for functions/modules, `CamelCase` for types.
- CLI flags use `kebab-case` (for example, `--shell-type`, `--socket`).
- Tests should use clear `test_*` names and mirror the feature being exercised.

## Testing Guidelines
- Frameworks: Rust test harness plus `rstest` for fixtures; integration tests live in `tests/integration.rs`.
- Integration tests create isolated tmux servers; keep them deterministic and ensure cleanup.
- Prefer adding tests alongside new tool behavior or security checks.

## Commit & Pull Request Guidelines
- Git history only shows "Initial commit", so no formal convention exists. Use concise, imperative subjects (example: "Add socket allowlist").
- PRs should include: a short summary, testing performed, and any new env vars or config.
- Link related issues and include screenshots only for user-facing CLI output changes.

## Security & Configuration Tips
- Security policy schema lives in `src/security.rs`; users supply a `config.toml` (see README sample). Keep defaults safe and explicit.
- Use isolated tmux sockets for agents via `--socket` or `TMUX_MCP_SOCKET`; use `--ssh` or `TMUX_MCP_SSH` for remote control.

---
> Source: [bnomei/tmux-mcp](https://github.com/bnomei/tmux-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
