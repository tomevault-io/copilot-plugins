## mazessh

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
# Development (starts Vite + Rust backend with hot reload)
pnpm tauri dev

# Production build (Windows: generates MSI + NSIS installers)
pnpm tauri build

# Frontend only (Vite dev server on port 1420)
pnpm dev

# Rust backend only
cargo build --manifest-path src-tauri/Cargo.toml

# Run Rust tests
cargo test --manifest-path src-tauri/Cargo.toml

# TypeScript type check (note: tsc cannot resolve @/ aliases, use for reference only)
pnpm tsc --noEmit

# Generate app icons from source image (must be square PNG)
pnpm tauri icon path/to/image.png
```

## Architecture

**Tauri 2 desktop app** with Rust backend and React + TypeScript frontend. The app manages SSH identities for Git workflows — switching profiles, injecting SSH keys into the system agent, and syncing git config.

### Backend (Rust) — `src-tauri/src/`

**State** uses two separate `Mutex`-wrapped structs to avoid contention:
- `AppState.inner` — profiles, active_profile_id, repo_mappings
- `AppState.security` — lock state, PIN status, activity timestamps, settings

**Command pattern**: All Tauri commands live in `commands/` modules. Protected commands call `ensure_unlocked(&state)?` as the first line, which checks lock state and resets the inactivity timer. Commands that must work while locked: `verify_pin`, `get_lock_state`, `lock_app`, `touch_activity`.

**Activation flow** (`commands/switch.rs`):
1. Fast path (blocks UI briefly): update state + save to disk + write env file
2. Background (`tokio::task::spawn_blocking`): start SSH agent, `ssh-add` key, set Windows env var, sync git identity
3. Emit `agent-status` event to frontend when done

**Background timer** (`tauri::async_runtime::spawn` in `lib.rs` setup): runs every 15 seconds, calls `session_service::check_inactivity_and_lock()` and `check_agent_expiry()`. Do NOT use `tokio::spawn` in Tauri setup — it panics because there's no reactor.

**Persistence** — all JSON files at `~/.maze-ssh/`:
- `profiles.json`, `active.txt`, `repo_mappings.json`, `settings.json`, `env` (shell-sourceable)
- `audit.log` (JSONL append-only)
- Secrets (PIN hash, passphrases) in Windows Credential Manager via `keyring` crate

**SSH config management** (`services/config_engine.rs`): uses marker-based sections (`BEGIN/END MAZE-SSH MANAGED`) in `~/.ssh/config`. Never parses existing config — only replaces content between markers.

### Frontend (React) — `src/`

**Path alias**: `@/` maps to `src/` via Vite's `resolve.alias`. The `paths` entry in `tsconfig.json` exists for IDE support but `tsc` cannot resolve it without `baseUrl` (deprecated). The build script uses `vite build` only, not `tsc && vite build`.

**Stores** (Zustand): `profileStore`, `appStore`, `logStore`, `themeStore`, `securityStore`, `repoMappingStore`. Each store wraps Tauri invoke calls from `lib/tauri-commands.ts`.

**Event flow**: Backend emits events (`agent-status`, `lock-state-changed`, `agent-expired`) via `app.emit()`. Frontend listens in `App.tsx` via `@tauri-apps/api/event`.

**Lock screen**: `LockScreen` component renders as a fixed overlay at `z-[100]` when `securityStore.isLocked` is true. It captures all keyboard events to prevent bypass.

**Custom titlebar**: `decorations: false` in tauri.conf.json. `TitleBar.tsx` implements Windows 11-style controls. Window operations require permissions in `capabilities/default.json` (e.g., `core:window:allow-minimize`).

**Theme**: CSS variables in `globals.css` with `.dark` / `.light` class on `<html>`. `themeStore` persists to `localStorage` and applies class on load.

### Key Patterns

- **Windows SSH**: Uses `C:\Windows\System32\OpenSSH\ssh.exe` and `ssh-add.exe`. The `ensure_agent_running()` function starts the Windows OpenSSH Agent service via PowerShell, with UAC elevation fallback.
- **Git operations**: Uses `std::process::Command` to call `git` binary. On Windows, prefers `C:\Program Files\Git\cmd\git.exe`.
- **`.gitignore` caution**: Use `/logs/` not `logs/` — the latter matches `src/components/logs/` and prevents tracking.
- **Cross-platform builds**: GitHub Actions workflow at `.github/workflows/release.yml`, triggered by `v*` tags. Uses `tauri-apps/tauri-action@v0`.

## Data Flow for Profile Activation

```
User clicks "Activate" in UI
  → appStore.activateProfile(id)
    → invoke("activate_profile", { id })
      → [Rust] ensure_unlocked → update state → save active.txt → write env file
      → [Rust background] set_user_env_git_ssh_command → ensure_agent_running → agent_switch_key → set_git_identity_global
      → emit("agent-status", { status, success })
    → [Frontend] update tray tooltip, re-fetch profiles
  → [Frontend] listen("agent-status") → toast + log
```

---
> Source: [khanhnd157/mazessh](https://github.com/khanhnd157/mazessh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
