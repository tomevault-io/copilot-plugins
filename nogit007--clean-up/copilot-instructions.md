## clean-up

> Interactive macOS cleanup tool — Tauri v2 desktop app.

# Clean Up

Interactive macOS cleanup tool — Tauri v2 desktop app.

## Architecture

- **Backend (Rust)**: `src-tauri/src/` — Tauri v2 app with IPC commands
  - `lib.rs` — app setup, command registration, appicon:// protocol
  - `commands.rs` — 11 Tauri IPC commands (scan, trash, permissions, etc.)
  - `types.rs` — shared types with serde camelCase serialization
  - `scanners/` — 6 scanners (dev-artifacts, system-caches, app-leftovers, large-old-files, unused-apps, homebrew-cleanup)
  - `utils/` — fs helpers, trash (osascript + safety blocklist), app detection (mdfind/mdls)
- **Frontend**: `frontend/index.html` — single-file UI using Tauri IPC (`invoke()`)
- **Config**: `src-tauri/tauri.conf.json` — Tauri v2 configuration
- **Build**: `scripts/build.sh` — Tauri build pipeline + icon generation
- **Install**: `scripts/install.sh` — copies to ~/Applications, Spotlight indexing

## Critical Rules

- Never use `rm` — all deletions go through trash safety blocklist in `src-tauri/src/utils/trash.rs`
- Path blocklist (BLOCKED_PATHS + BLOCKED_PREFIXES) must never be weakened
- Version must be bumped in: `package.json`, `src-tauri/Cargo.toml`, `src-tauri/tauri.conf.json`

## Development

- Build: `cargo tauri build` (from project root)
- Test: `cargo test` (from `src-tauri/`)
- Dev: `cargo tauri dev` (from project root)

## Detailed Rules

See `.claude/rules/` for context-specific rules:

- `workflow.md` — version bumping, release notes, push protocol

---
> Source: [NOGIT007/clean-up](https://github.com/NOGIT007/clean-up) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
