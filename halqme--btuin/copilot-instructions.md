## btuin

> - `src/`: TypeScript source for the `btuin` library. Public entry points live in `src/index.ts` and `src/**/index.ts`.

# Repository Guidelines

## Project Structure & Module Organization

- `src/`: TypeScript source for the `btuin` library. Public entry points live in `src/index.ts` and `src/**/index.ts`.
- `src/layout-engine/`: Rust layout engine (FFI) consumed by `src/layout-engine/index.ts`.
- `tests/`: Bun test suite (`bunfig.toml` sets `tests/` as the test root).
  - `tests/units/`: focused unit tests by area (`renderer/`, `runtime/`, `terminal/`, etc.)
  - `tests/integration/`: cross-module behavior tests
  - `tests/e2e/`: end-to-end runs and package verification
- `docs/`: Markdown documentation.
- `examples/`: runnable examples (e.g., `examples/counter.ts`).
- `scripts/`: profiler/perf regression specs (run via Bun test runner).

## Build, Test, and Development Commands

This repo uses `mise` to pin toolchains (see `mise.toml`).

- `mise install`: install pinned tool versions (Bun, Rust, oxc tools).
- `mise exec -- bun install --frozen-lockfile`: install JS dependencies.
- `mise run build:ffi`: build the Rust FFI binary (`src/layout-engine/target/release`).
- `mise run test` / `mise run test:watch`: run tests (Bun’s test runner).
- `mise run format` / `mise run lint` / `mise run check`: format (oxfmt), lint (oxlint), and type-check (`bunx tsc --noEmit`).
- `mise run precommit`: run format + autofix lint + type-check before pushing.

## Coding Style & Naming Conventions

- TypeScript (ESM) with strict type-checking (`tsconfig.json`).
- Prefer small, composable modules and re-export via `src/**/index.ts`.
- Use lowercase filenames; use `kebab-case` for multi-word files (e.g., `render-loop.ts`).
- Run `mise run test && mise run precommit` after any edit.
- Run `mise run format` before opening a PR; let `oxfmt` decide whitespace.
- This project targets Bun-native development; optimizing to remove Bun dependencies from the core package is not a goal.

## Testing Guidelines

- Use Bun’s test runner (`bun test` via `mise run test`).
- Naming conventions in this repo include `*.test.ts`, `*.spec.ts`, and `*.integration.test.ts`; follow the closest existing pattern in the folder you’re editing.
- If you change the layout engine or its JS bindings, run `mise run build:ffi` before `mise run test`.

## Commit & Pull Request Guidelines

- Commit messages in history are short and imperative (e.g., “Fix …”, “Update …”); keep the subject concise and scoped (optionally prefix with an area like `renderer:`).
- PRs should include: what changed, why, how to test (exact commands), and any UX/output evidence for TUI changes (screenshot/recording if applicable).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HALQME) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
