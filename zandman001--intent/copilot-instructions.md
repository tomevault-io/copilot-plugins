## intent

> - Dev: `npm run tauri:dev` (starts Vite + Rust backend)

# Agent Guidelines for Intent Synth

## Build & Run
- Dev: `npm run tauri:dev` (starts Vite + Rust backend)
- Build: `npm run tauri:build`
- Frontend only: `cd frontend && npm run dev`
- Rust check: `cd src-tauri && cargo check`
- Rust tests: `cd src-tauri && cargo test` (single test: `cargo test test_name`)

## Code Style
**Rust:** Standard rustfmt (4-space indent). Imports: std → external crates → local. Use `anyhow::Result` for errors. Lock-free audio: use `Param`/`ParamHandle` atomics, never `Mutex` in DSP hot path. Module structure: `pub mod name; use crate::module::Type;`

**TypeScript:** Strict mode enabled. Functional components + hooks only. Imports: React → external → local → CSS. Type everything (no `any`). Props as inline types or `type Props = {...}`. Errors: try/catch with state updates in useIPC hook.

**Naming:** camelCase (TS/JS), snake_case (Rust). Components PascalCase. CSS: kebab-case classes, --custom-properties.

**Style:** 1-bit brutalist aesthetic (strict #000/#fff). Pixel-perfect 8px grid. No rounded corners, no shadows. Tiny5 font uppercase. All measurements divisible by 8.

## Architecture
UI (React) ↔ Tauri IPC ↔ Rust backend ↔ lock-free channels ↔ audio thread. Never block audio thread. State flows: UI → backend via `invoke()`, backend → UI via `emit()` events. AI: JSON-only responses, handle both Ollama/OpenAI.

---
> Source: [Zandman001/Intent](https://github.com/Zandman001/Intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
