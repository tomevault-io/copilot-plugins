## balatro-mod-manager

> - `src/`: SvelteKit UI (components, routes, stores, utils).

# Repository Guidelines

## Project Structure & Module Organization

- `src/`: SvelteKit UI (components, routes, stores, utils).
- `src-tauri/`: Tauri app and Rust backend (`lib.rs`, commands, config).
- `src-tauri/bmm-lib/`: Core Rust library used by the app.
- `static/` and `images/`: Static assets bundled at build.
- `scripts/`, `.github/`, `build/`: Helper scripts, CI, and build output.
- `tests/`: Shared Rust test helpers; crate-specific tests live next to code.

## Build, Test, and Development Commands

- Dev (desktop): `task debug` or `bun run tauri dev` — launches the Tauri app.
- Dev (web only): `bun run dev` — Vite dev server for UI.
- Build (desktop): `task release` or `bun run tauri build` — production bundle.
- Preview (web): `bun run preview` — serves built UI.
- Type check: `bun run check` — Svelte/TS checks.
- Rust tests: `cargo test` in `src-tauri/` and `src-tauri/bmm-lib/`.
- Clean: `task clean` — removes build artifacts.

## Coding Style & Naming Conventions

- Svelte/TS: 2-space indent; components `PascalCase.svelte`; stores/utils `camelCase.ts`.
- Routing: follow SvelteKit `+page.svelte`, `+layout.svelte` patterns.
- Rust: `snake_case` for modules/functions, `CamelCase` for types; prefer `Result` over panics.
- Format: run `cargo fmt` (Rust); use editor formatting for Svelte/TS; keep imports ordered.

## Testing Guidelines

- Rust unit tests live beside code (`#[cfg(test)]`); integration tests per crate; use `rstest` when helpful.
- Name tests with descriptive `snake_case`; focus on core logic and file/IO boundaries.
- Frontend: no formal unit tests; use `bun run check` and manual flows for regressions.
- Run: `cargo test -q` in `src-tauri/` and `src-tauri/bmm-lib/` before PRs.

## Commit & Pull Request Guidelines

- Commits: imperative mood; small, focused. Conventional prefixes allowed (e.g., `feat:`, `fix:`, `chore:`).
- PRs: clear description, linked issues, test steps, platforms tested (Windows/macOS), and screenshots/GIFs for UI changes.
- Keep changes scoped; update `README.md` or in-app help if behavior changes.

## Security & Configuration Tips

- Do not commit secrets or signing identities. macOS signing is configured in `Taskfile.yml` for local use only.
- Ensure Rust toolchain, Bun, and Tauri prerequisites are installed for your OS.

---
> Source: [skyline69/balatro-mod-manager](https://github.com/skyline69/balatro-mod-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
