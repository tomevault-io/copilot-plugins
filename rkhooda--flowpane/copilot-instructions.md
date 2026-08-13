## flowpane

> Developer Brain lives at: `/Users/rkhooda/Documents/Rkxee Obsidian/Developer's brain`

# FlowPane — Claude Operating Manual

## Developer Brain

Developer Brain lives at: `/Users/rkhooda/Documents/Rkxee Obsidian/Developer's brain`

At the start of every session, read in this order:

1. `/Users/rkhooda/Documents/Rkxee Obsidian/Developer's brain/CLAUDE.md` — governing operating principles
2. `/Users/rkhooda/Documents/Rkxee Obsidian/Developer's brain/ARCHITECTURE.md` — zone structure and content boundaries
3. `/Users/rkhooda/Documents/Rkxee Obsidian/Developer's brain/projects/flowpane/overview.md` — project context in Developer Brain (read if the file exists; skip if not)

The principles in Developer Brain govern every session in this repository. Do not duplicate them here. If anything here conflicts with Developer Brain, follow Developer Brain and flag the conflict.

---

## Project

**Name:** `flowpane`
*Must match the folder name at `Developer Brain/projects/flowpane/`.*

**What it is:** A frameless, always-on-top floating task manager for desktop — built with Tauri 2, featuring edge-snap/collapse behavior, focus mode with timer, and system tray integration.

**Stack:** Rust (Tauri 2 backend, `src-tauri/src/lib.rs`), Vanilla JS/HTML/CSS (frontend, `src/`), `tauri-plugin-store` for persistent storage

---

## Rules

**Running the app:**
```
npm run tauri dev      # development
npm run tauri build    # production build
```

**Architecture:**
- All window management, IPC commands, and state live in `src-tauri/src/lib.rs`
- Frontend (`src/`) is vanilla JS — no build step, no framework, no bundler
- Tauri IPC via `window.__TAURI__.core.invoke()`; Tauri events via `window.__TAURI__.event`
- Window state (hover tracking, active window, spawn order) is managed server-side in Rust `State<>`

**Constraints:**
- Max 8 concurrent app windows (`MAX_APP_WINDOWS`)
- Window spawn rate-limited to 700ms (`MIN_WINDOW_SPAWN_INTERVAL`)
- Window dimensions are fixed: 335×405px (`APP_WINDOW_WIDTH/HEIGHT`)
- Always-on-top, frameless, transparent — do not change these window properties

**Code style:**
- Rust: standard `cargo fmt` conventions; errors returned as `Result<_, String>`
- JS: no TypeScript. Native ES modules only (`<script type="module">`; `import`/`export` between `src/js/*.js`) — no bundler, no build step
- Do not introduce a frontend framework or build toolchain

---

## Session End

At the end of any significant session:

1. Has the project's technical state changed substantially? → Update `Developer Brain/projects/flowpane/overview.md`
2. Was a significant architectural decision made? → Create `Developer Brain/projects/flowpane/decision-[topic].md`
3. Does any knowledge from this session pass the promotion criteria in `Developer Brain/WORKFLOW.md`? → Promote it

Do not update Developer Brain for routine task completion or information visible from the code.

---
> Source: [rkhooda/FlowPane](https://github.com/rkhooda/FlowPane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
