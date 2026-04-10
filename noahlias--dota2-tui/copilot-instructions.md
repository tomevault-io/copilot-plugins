## dota2-tui

> - `src/`: Rust source code.

# Repository Guidelines

## Project Structure & Module Organization

- `src/`: Rust source code.
  - `src/api.rs`: OpenDota client, cache, rate limiting, logging.
  - `src/app.rs`: App state and message handling.
  - `src/ui/`: Ratatui layout, panels, tables, banner, images.
  - `src/i18n.rs`: All UI strings and labels.
- `examples/`: Example configs (see `examples/config.toml`).
- `tests/`: Integration tests (live API checks are opt-in).

Keep UI and data logic separate. Add new modules only when a file gets too large.

## Build, Test, and Development Commands

- `cargo build`: Compile the project.
- `cargo run`: Run the TUI locally.
- `cargo test`: Run all tests.
- `cargo fmt`: Apply Rust formatting.
- `cargo clippy`: Lint for common mistakes.

If you add feature flags, document them in this section (e.g., `cargo run --features mock-api`).

## Coding Style & Naming Conventions

- Indentation: 4 spaces (Rust default).
- Prefer `snake_case` for functions/modules and `UpperCamelCase` for types.
- Keep async boundaries explicit (`async fn` + `tokio`), and avoid blocking calls on the async runtime.

Use `rustfmt` and `clippy` before pushing.

## Testing Guidelines

- Use Rust’s built‑in test framework (`cargo test`).
- Name tests by behavior (e.g., `fetch_user_profile_success`).
- Prefer small, fast tests. Live API checks must be opt‑in using `OPENDOTA_LIVE=1`.

## Commit & Pull Request Guidelines

- This repo is early-stage; if no conventions are established yet, use Conventional Commits (e.g., `feat: add match list view`).
- PRs should describe the change, mention any API endpoints touched, and include screenshots or terminal captures for UI changes when relevant.

## Configuration & Platform Notes

- Store user config in OS‑specific directories:
  - Linux: `~/.config/dota2_tui/`
  - macOS: `~/Library/Application Support/dota2_tui/`
  - Windows: `%APPDATA%\\dota2_tui\\`
- Logs (when enabled) are written to `tui.log` in the config directory.
- Recent searches append to `recent.jsonl` in the config directory.
- Image cache is stored under the OS cache dir (`dota2_tui/images`).
- Document required API keys and environment variables in `config/` or `README` once added.

## Agent Notes

- Keep diffs minimal and remove unused code and imports.
- Only add comments for non‑obvious, persistent logic.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/noahlias)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/noahlias)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
