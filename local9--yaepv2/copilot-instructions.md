## yaepv2

> description: Expert Tauri 2 + Rust backend for native desktop commands, events, and services

---
description: Expert Tauri 2 + Rust backend for native desktop commands, events, and services
globs: src-tauri/**/*.rs
alwaysApply: false
---

# Tauri 2 and Rust (expert defaults)

Act as an expert in **Tauri 2** (commands, events, capabilities, webview/window lifecycle, IPC contracts) and **Rust** (ownership/borrowing, `Send`/`Sync` across threads, idiomatic error handling, minimal `unsafe`, structured logging).

- Keep native responsibilities in Rust: global hotkeys, DWM thumbnails, native window lifecycle, focus tracking, and process scanning.
- Expose small, typed command surfaces; keep request/response types serializable and stable; validate every input at the command boundary.
- Prefer explicit `Result` and error enums (`thiserror` or equivalent) over stringly errors; return messages the UI can show or map.
- Organize backend by domain modules: `db`, `hotkeys`, `thumbnail_service`, `windows`, `dwm`, `eve_profile_tools`.
- Avoid blocking the async runtime; offload heavy or blocking work appropriately and document shared state (`Arc`, mutexes, channels).
- Persist settings in **SQLite**; use migrations and additive schema changes.
- Document `unsafe` and Win32 interop briefly at the callsite; keep `unsafe` blocks as small as possible.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Local9) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
