## skilldock

> SkillDock is a desktop app for managing a local collection of agent skills, built with React, Vite, TypeScript, and Tauri.

# Repository Guidelines

## Project Structure & Module Organization

SkillDock is a desktop app for managing a local collection of agent skills, built with React, Vite, TypeScript, and Tauri.

- `src/` contains the frontend. `src/App.tsx` holds the main UI, `src/main.tsx` boots React, `src/lib/commands.ts` wraps Tauri calls, and `src/lib/types.ts` defines shared frontend types.
- `src/styles.css` contains global styling for the app.
- `src-tauri/src/` contains Rust backend modules for scanning, config storage, links, git operations, tasks, and agent profiles.
- `src-tauri/tests/` contains Rust integration tests grouped by backend capability.
- `scripts/` contains utility checks; `docs/` contains product and implementation notes.
- `dist/`, `node_modules/`, and `src-tauri/target/` are generated outputs; do not edit them manually.

## Build, Test, and Development Commands

- `npm run dev` starts the Vite frontend on `127.0.0.1`.
- `npm run tauri:dev` starts the full Tauri desktop app with the `desktop` feature enabled.
- `npm run build` runs TypeScript checking and builds the frontend bundle.
- `npm run tauri:build` builds the packaged Tauri app.
- `npm run ui:smoke` renders key React views through Vite SSR and checks expected UI text.
- `npm run rust:test` runs `cargo test --manifest-path src-tauri/Cargo.toml`.

Run frontend and Rust checks before a PR when touching shared command contracts.

## Coding Style & Naming Conventions

Use TypeScript with React function components and hooks. Keep types explicit where data crosses the Tauri boundary. Follow the existing two-space indentation in `src/`. Use `camelCase` for variables/functions, `PascalCase` for components/types, and kebab-like names only for user-facing IDs or paths.

For Rust, use `rustfmt`, `snake_case` module/function names, and small modules aligned to backend capabilities. Keep serialization structs compatible with `src/lib/types.ts`.

## Testing Guidelines

Rust integration tests live in `src-tauri/tests/` and should be named for behavior, for example `workspace_selection.rs` or `link_operations.rs`. Update these tests when changing backend behavior.

Use `npm run ui:smoke` for broad frontend regression coverage. Add assertions when changing major views, command fallback behavior, or rendered labels.

## Commit & Pull Request Guidelines

The full workflow — branch prefixes, Conventional Commits, required CI checks, rebase-merge, and the release procedure — lives in [`CONTRIBUTING.md`](CONTRIBUTING.md) (or [`CONTRIBUTING.zh-CN.md`](CONTRIBUTING.zh-CN.md)). Follow it rather than inventing new conventions; it matches the repo's branch-protection configuration.

Short version: branch off `main`, run the local gates before pushing, open a PR, wait for all four CI checks to go green, then `gh pr merge <n> --rebase --delete-branch`.

---
> Source: [yiwen65/SkillDock](https://github.com/yiwen65/SkillDock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
