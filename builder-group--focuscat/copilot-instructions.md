## focuscat

> 1. **Think first, then act** - Read the codebase for relevant files before making changes.

# Codex Instructions

## Behavior

1. **Think first, then act** - Read the codebase for relevant files before making changes.

2. **Check in before major changes** - Before making any significant changes, check in with me and I will verify the plan.

3. **Explain as you go** - Every step of the way, give me a high-level explanation of what changes you made.

4. **Keep it simple (KISS)** - Make every task and code change as simple as possible. Avoid massive or complex changes. Every change should impact as little code as possible. Everything is about simplicity.

5. **No speculation** - Never speculate about code you have not opened. If I reference a specific file, you MUST read it before answering. Investigate and read relevant files BEFORE answering questions about the codebase. Never make claims about code before investigating unless certain - give grounded, hallucination-free answers.

## Project

Tauri desktop apps for productivity and activity tracking.

**Tech Stack:**

- Rust + Swift (backend, `crates/`)
- TypeScript + React (frontend, `apps/`)
- SQLite (database)
- tauri-specta (type-safe IPC)

**Key Commands:**

- `pnpm start:dev` - Start development
- `pnpm build` - Build all
- `pnpm test` - Run all tests
- `cargo test -p mado` - Test mado crate

**Important Directories:**

- `crates/mado/` - Window monitoring library (Swift + Rust FFI)
- `apps/desktop/` - FocusCat desktop app (pomodoro + cat widget)
- `apps/web/` - FocusCat and PomodoroCat website (focuscat.app + pomodorocat.com)

---
> Source: [builder-group/focuscat](https://github.com/builder-group/focuscat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
