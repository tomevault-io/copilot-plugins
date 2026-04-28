## tend

> Local agent orchestration UI for human-in-the-loop automation. MVP complete (155/155 tasks across 9 phases).

# tend

Local agent orchestration UI for human-in-the-loop automation. MVP complete (155/155 tasks across 9 phases).

## What It Does

Desktop workbench that unifies AI agent sessions across multiple repositories into a single view with live status, needs-input alerts, companion terminals, per-project scratchpads, and automatic workspace persistence.

## Architecture

```text
tend/
  protocol/       # Shared daemon IPC wire types (serde enums) — tend-protocol
  src-tauri/      # Rust backend: PTY management, SQLite, daemon IPC, Tauri commands — tend-workbench
  cli/            # CLI wrapper: `tend run` — launches agents under PTY, registers with workbench — tend-cli
  src/            # Svelte 5 frontend: session list, split view, scratchpad, alerts
  tests/e2e/      # Playwright E2E specs
  specs/          # Feature specifications and planning docs
```

### Workspace Crates

| Crate | Purpose | Publishable |
|-------|---------|-------------|
| `tend-protocol` | IPC wire format shared by workbench + CLI | No |
| `tend-workbench` | Tauri 2 desktop app + Rust backend | No (OS binary) |
| `tend-cli` | `tend run` CLI entry point | No |

## Tech Stack

- **Backend**: Rust 2024 edition (≥1.89), Tauri 2, `portable-pty`, `sqlx` (SQLite), `tokio`, `notify`, `sysinfo`, `tracing`
- **Frontend**: TypeScript 5.x, Svelte 5 runes, xterm.js 5 (`fit` + `web-links` addons), `marked`, `DOMPurify`, Vite
- **Testing**: `cargo test` (188 Rust tests), `vitest` (frontend), Playwright (E2E), `cargo deny` (license/dep gating)

## Commands

```bash
# Dev
pnpm install && cargo fetch
pnpm tauri dev

# Backend tests + lint
cargo test
cargo clippy --workspace -- -D warnings
cargo fmt --check
cargo deny check bans

# Frontend
pnpm check          # svelte-check type checking
pnpm test           # vitest

# E2E (requires tauri-driver)
pnpm e2e
```

## Code Style

- Rust: edition 2024, `unsafe` blocks required for `set_var`/`remove_var`/`getuid`, `unsafe extern "C"` for FFI
- TypeScript/Svelte: Svelte 5 runes API, standard conventions
- Tauri commands use `snake_case` args (must match Rust arg struct fields)
- Workspace resolver v3, Cargo.lock v4

## Key Patterns

- Sessions managed as per-session actor tasks on Tokio
- PTY spawning via `portable-pty`
- Daemon IPC over Unix-domain socket with length-prefixed JSON (tend-protocol)
- SQLite via `sqlx` with compile-time query checking
- Frontend stores for reactive state; Tauri `invoke()` for backend calls

## Known v1 Limitations

- Sessions started without `tend run` won't appear in the workbench
- No remote session support (SSH, cloud workstations, WSL-to-Windows)
- No VS Code / editor companion — terminal only
- Scratchpads are plain text with light markdown; no images or attachments
- Activity summary uses ring-buffer last-line heuristic; no LLM summarization
- Workbench-owned sessions become read-only mirrors after workbench restart

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/knitli) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
