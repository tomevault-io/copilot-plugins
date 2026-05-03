## codex-trace

> npm run dev              # Vite dev server

# CLAUDE.md

## Commands

```bash
# Dev
npm run dev              # Vite dev server
npm run tauri dev        # Full Tauri desktop app
npm run dev:web          # Web mode (opens browser)

# Lint
npx oxlint              # JS/TS lint
cargo clippy --manifest-path src-tauri/Cargo.toml  # Rust lint

# Format
npx oxfmt               # JS/TS format
cargo fmt --manifest-path src-tauri/Cargo.toml     # Rust format

# Test
npx vitest run           # Frontend tests
cargo test --manifest-path src-tauri/Cargo.toml    # Rust tests

# Type check
npx tsc --noEmit

# All at once
npm run check            # tsc + oxlint + oxfmt --check + clippy + cargo fmt --check + vitest + cargo test
```

## Rule

After every code change (src, tests, config that affects build), always add enough tests for the changes, then run lint, format, and test before committing:

```bash
npx oxfmt && npx oxlint && npx tsc --noEmit && cargo fmt --manifest-path src-tauri/Cargo.toml && cargo clippy --manifest-path src-tauri/Cargo.toml -- -D warnings && cargo test --manifest-path src-tauri/Cargo.toml
```

## Architecture

- **Backend:** Rust + Tauri v2 + axum HTTP server (port 11424)
- **Frontend:** React 19 + TypeScript + Vite
- **Sessions:** `~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl`

### Key files

- `src-tauri/src/parser/` — JSONL parsing pipeline
  - `entry.rs` — raw line parsing, format detection
  - `discover.rs` — session discovery + metadata scan
  - `session.rs` — full session parse
  - `turn.rs` — turn boundary detection (new + old format)
  - `toolcall.rs` — tool call classification by end event
- `src-tauri/src/http_api.rs` — axum routes (port 11424)
- `src/App.tsx` — 3-view state machine (picker → list → detail)
- `src/components/SidebarTree.tsx` — CRITICAL: date-grouped JSONL folder structure
- `shared/types.ts` — TypeScript types (must match Rust structs)

### JSONL format

Three `session_meta` variants (new/mid/oldest). Turn boundary detection uses
`task_started`/`task_complete` for newer CLI; `user_message` boundaries for older.
Tool calls classified by **end event type**, not function name.

### Ports

- Frontend dev: 1420
- Backend HTTP: 11424
- Docker: 1422

---
> Source: [PixelPaw-Labs/codex-trace](https://github.com/PixelPaw-Labs/codex-trace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
