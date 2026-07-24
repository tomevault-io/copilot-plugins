## embedist

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo layout note

The app root is this directory (`embedist/embedist`). The parent folder `embedist/` is only a container — do not run npm/cargo from there. A separate Docusaurus package lives in `docs/` with its own `package.json`; only run its scripts when editing docs.

`AGENTS.md` in this directory contains overlapping developer notes; keep both in sync when behavior changes.

## Commands

Frontend scripts (`package.json`):
- `npm run dev` — Vite dev server (port 1420, required by `tauri.conf.json`)
- `npm run build` — `tsc && vite build` (runs TypeScript strict-mode check)
- `npm run tauri dev` — run the full desktop app in development
- `npm run tauri build` — produce release artifacts

There is **no** `lint` or `test` npm script. No automated test suite exists (`*.test.*`, `*.spec.*`, `src-tauri/tests` are all absent). Validation = `npm run build` + `cargo clippy --manifest-path src-tauri/Cargo.toml` + manual app checks.

Release artifacts from `npm run tauri build`:
- Portable: `src-tauri/target/release/embedist.exe`
- Installer: `src-tauri/target/release/bundle/nsis/Embedist_<version>_x64-setup.exe` (NSIS only — set by `tauri.conf.json` `bundle.targets`)

## Release version sync

When bumping the version, update **all three** files — they're independent and drift silently:
- `package.json`
- `src-tauri/Cargo.toml`
- `src-tauri/tauri.conf.json`

## Architecture

Tauri 2 desktop app (Windows-only target). React 18 + TypeScript (strict) frontend talks to a Rust backend via Tauri `invoke` commands.

**Frontend entry:** `src/main.tsx` → `src/App.tsx`. State lives in Zustand stores under `src/stores/` (`aiStore`, `fileStore`, `settingsStore`, `uiStore`) with `localStorage` persistence. Business logic is in hooks (`src/hooks/use*.ts`).

**Backend entry:** `src-tauri/src/main.rs` → `src-tauri/src/lib.rs`. All Tauri commands are registered in the `invoke_handler!` macro in `lib.rs` and implemented in `src-tauri/src/commands/{ai,filesystem,platformio,pty,serial}.rs`. Shared async state (`SerialState`, `AIState`, `BuildState`, `PtyState`, `WatchState`) is attached via `.manage()` in `lib.rs` — add new state there.

**AI modes:** chat / plan / agent / debug. Mode-specific system prompts are Markdown files at `src/lib/prompts/modes/*.md`, imported as `?raw` strings via `src/lib/prompts/index.ts`. `PROMPTS` in that file also defines each mode's RAG context categories and empty-state copy.

**Tool calling:** Agent-mode tool execution runs client-side — `src/hooks/useAgent.ts` drives the loop (up to `MAX_ITERATIONS = 50`), dispatching to tools in `src/lib/agent-tools.ts`. Path safety checks live in `useAgent.ts` (`isPathSafe`, `PROJECT_SCOPED_TOOLS`). The Rust backend (`src-tauri/src/commands/ai.rs`) implements tool-enabled chat for OpenAI / Anthropic / DeepSeek / custom OpenAI-compatible endpoints. **Ollama and Google providers are text-only** in the backend — don't assume tool calling works there.

**PlatformIO integration:** `src-tauri/src/commands/platformio.rs` shells out to `pio`. Build output streams via Tauri events; errors are parsed into a Problems panel. Build cancellation uses the shared `BuildState`.

**CSP:** `tauri.conf.json` restricts `connect-src` to the specific AI provider hosts + `localhost:11434` (Ollama). Adding a new provider host means updating that CSP.

## Rust gotchas (from prior pain)

- For cloneable async shared state, use `Arc<tokio::sync::Mutex<...>>` (see `BuildState` in `src-tauri/src/commands/platformio.rs`).
- Do **not** use `Option::is_none_or(...)` — prefer `is_some_and(...)` or explicit match for stable Rust.
- `run_shell` in `src-tauri/src/commands/filesystem.rs` rejects shell metacharacters; it won't run chained/piped expressions.

---
> Source: [mandarwagh9/embedist](https://github.com/mandarwagh9/embedist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
