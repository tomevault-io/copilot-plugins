## clawbox

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ClawBox is a cross-platform desktop application (Tauri v2) for managing AI coding agents: it detects/installs agents (Claude Code, Codex, OpenClaw, etc.), configures model providers, and syncs provider/MCP/skills/memory settings down to each agent's own config files.

## Architecture

### Backend (Rust, `src-tauri/`)
- Entry: `src-tauri/src/main.rs` → `lib.rs` (`run()` registers all Tauri commands via `invoke_handler`)
- `src/commands/` — Tauri command handlers exposed to the frontend:
  - `config.rs` — model provider / app config CRUD
  - `sync.rs` — sync orchestration (per-agent provider binding, overview)
  - `cc_switch.rs` — cc-switch import
  - `install.rs`, `agents.rs`, `aggregate.rs`, `provider_test.rs`
- `src/sync/` — per-agent config adapters (`providers.rs`, `codex.rs`, `skills.rs`, `memory.rs`, `json_file.rs`, `cli.rs`)
- `src/agents/`, `src/backends/` — agent detection and backend definitions
- `src/path_env.rs` — PATH bootstrapping (must run before threads spawn)

### Frontend (Svelte 5 + TypeScript + Vite, repo root)
- Entry: `src/main.ts` → `src/App.svelte`
- Pages in `src/routes/`: `agents/`, `providers/`, `mcp/`, `capabilities/`, `about/`
- `src/lib/api/` — typed wrappers around Tauri `invoke` calls
- `src/lib/stores/` — Svelte stores (e.g. `config.ts` for providers)
- `src/lib/i18n/` — `en.json` / `zh.json` locales (svelte-i18n)
- Styling: TailwindCSS 4 + Skeleton

Note: agent↔provider binding is per-agent (no global "star"/sync-all); the agent page selector has no unbind option — once bound, an agent stays managed by ClawBox.

## Commands

```bash
npm run dev          # frontend dev server (vite, port 1420)
npm run tauri dev    # full app in dev mode
npm run tauri build  # release build + bundles (.app / .dmg on macOS)
npm run check        # svelte-check type checking
cd src-tauri && cargo test   # Rust tests
```

Build artifacts land in `src-tauri/target/release/bundle/`.

## Key Files

- `src-tauri/tauri.conf.json` — app config (identifier `com.clawbox.desktop`, window, CSP)
- `src-tauri/src/sync/providers.rs` — per-agent provider adapters (largest module)
- `src/routes/agents/+page.svelte` — agent list, install/upgrade, provider binding UI
- `src/lib/i18n/{en,zh}.json` — all user-facing copy; keep both in sync

---
> Source: [zhaolianghz/clawbox](https://github.com/zhaolianghz/clawbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
