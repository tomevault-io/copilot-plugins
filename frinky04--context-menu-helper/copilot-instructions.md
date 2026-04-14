## context-menu-helper

> - Run `cargo fmt` after Rust changes.

# AGENTS.md

## Task Completion Requirements

- Run `cargo fmt` after Rust changes.
- Run `cargo test -p context_menu_core` before marking work complete.
- Run `node --check ui/main.js` after frontend JavaScript changes.
- Do not claim desktop build verification unless tested on Windows (`cargo tauri dev`).

## Project Snapshot

Context Menu Helper is a Windows desktop tool for discovering, creating, enabling/disabling, and rolling back context-menu actions.

The codebase is intentionally split into:

- `core`: Rust domain engine for models, scanning, templates, apply/rollback, and persistence.
- `src-tauri`: Tauri desktop shell exposing `core` functionality via `tauri::command` handlers.
- `ui`: static HTML/CSS/JS frontend loaded by Tauri.

## Core Priorities

1. Safety and reversibility of registry changes.
2. Predictable behavior for non-technical users.
3. Maintainable architecture (shared logic in `core`, thin UI/shell layers).

If tradeoffs are needed, choose correctness + rollback safety over feature speed.

## Registry Safety Rules

- Prefer hide/disable semantics over destructive removal.
- Snapshot affected keys before applying changes.
- Keep write scope limited to approved shell paths.
- Treat shell extensions (`shellex`) as advanced behavior and surface clearly in UI.
- Never silently change precedence rules between `HKCU` and `HKCR`.

## Maintainability Rules

- Keep business rules in `core`; avoid duplicating registry logic in `src-tauri` or `ui`.
- UI should consume backend contracts, not invent separate state shapes without reason.
- Refactor repeated heuristics into helper functions/modules.
- Keep simple mode understandable for non-technical users; gate technical detail behind advanced mode.
- Keep implementation generic. Do not turn user-provided examples into product-specific labels, placeholders, helper copy, or test fixtures unless explicitly requested.

## Command Surface (Tauri)

The desktop app currently expects these backend commands to remain stable unless intentionally refactored:

- `scan_entries`
- `suggest_actions`
- `create_custom_entry`
- `toggle_entry`
- `apply_changes`
- `rollback`
- `list_change_sets`
- `get_change_set`

Any command rename or payload change requires synchronized updates in:

- `src-tauri/src/main.rs`
- `ui/main.js`
- relevant `core` model/service APIs

## Platform Notes

- `core` compiles/tests on non-Windows, but real registry behavior is Windows-only.
- Linux/WSL may fail Tauri desktop checks due to GTK/WebKit dependencies; do not treat that as a Windows failure.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/frinky04) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
