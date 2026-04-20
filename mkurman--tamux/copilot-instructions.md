## tamux

> This repository is a Rust workspace with a desktop frontend:

# Repository Guidelines

## Project Structure & Module Organization
This repository is a Rust workspace with a desktop frontend:
- `crates/amux-daemon`: core daemon (IPC server, session/state, policy, snapshots).
- `crates/amux-cli`: CLI client (`amux`) for daemon operations.
- `crates/amux-gateway`: chat platform bridge (Slack/Discord/Telegram routing).
- `crates/amux-mcp`: MCP JSON-RPC server.
- `crates/amux-protocol`: shared protocol/messages/config.
- `crates/amux-tui`: terminal UI for interactive daemon/session management.
- `frontend/`: React + TypeScript UI and Electron shell (`frontend/electron`, `frontend/src`).
- Build artifacts: `dist/`, `dist-release/`, `frontend/release/`.

## Build, Test, and Development Commands
Run from repo root unless noted:
- `cargo build --release`: build all Rust crates.
- `cargo run --release --bin amux-daemon`: start daemon.
- `cargo run --release --bin amux -- list`: basic CLI connectivity check.
- `cargo run --release --bin amux-tui`: launch the terminal UI.
- `cargo test --workspace`: run Rust unit tests.
- `cd frontend && npm install && npm run dev`: start frontend dev server.
- `cd frontend && npm run dev:electron`: launch Electron app.
- `cd frontend && npm run build:electron`: create production packages.
- `cd frontend && npm run lint`: run frontend lint pass.

## Coding Style & Naming Conventions
- Rust: follow `rustfmt` defaults (4-space indentation, snake_case modules/functions, PascalCase types).
- TypeScript/React: 2-space indentation, double quotes, semicolons; keep `strict` TypeScript compatibility.
- Component files use PascalCase (example: `SystemMonitorPanel.tsx`).
- Store/util files use camelCase with explicit suffixes (example: `agentMissionStore.ts`, `sessionPersistence.ts`).
- Keep crate boundaries clean: shared wire types belong in `amux-protocol`, not duplicated in app crates.
- For TUI work, keep rendering, input handling, and state/update logic in separate focused modules when practical.
- Strong rule: every newly created file must stay under 500 lines of code. Split features into smaller modules/components before a file reaches 500 LOC.
- **CRITICAL - NO SHORTCUTS**: Never mock, stub, or placeholder any function with intent to complete later. Always implement fully or decline the task explicitly. Violation results in immediate rejection and conversation termination.

## Testing Guidelines
- Prefer unit tests close to implementation in Rust (`#[cfg(test)] mod tests` in source files).
- Name tests by behavior (example: `run_prefix_routes`).
- For TUI changes, run the TUI against a live daemon or equivalent local setup and smoke-test the affected flows manually.
- No formal frontend test suite is committed yet; for UI changes, validate with `npm run lint`, `npm run build`, and manual Electron smoke checks.
- No coverage threshold is enforced currently; add tests for any new parsing, routing, policy, or state logic.

## Commit & Pull Request Guidelines
- Recent history favors Conventional Commit prefixes: `feat:`, `refactor:`; keep subjects imperative and specific.
- Keep commits scoped (one concern per commit) and avoid mixing Rust backend and UI refactors without need.
- PRs should include:
  - concise change summary and rationale,
  - test/lint commands run and results,
  - linked issue/task,
  - screenshots or short recordings for UI behavior changes.

---
> Source: [mkurman/tamux](https://github.com/mkurman/tamux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
