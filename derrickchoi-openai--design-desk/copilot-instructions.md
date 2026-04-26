## design-desk

> - `src/`: Main frontend (React + TypeScript).

# Repository Guidelines

## Project Structure & Module Organization
- `src/`: Main frontend (React + TypeScript).
- `server/`: Local collaboration/WebSocket server run during dev.
- `docs/`: Design notes, milestones, and reference material.
- `scripts/`: Utility scripts (for example, export tooling).
- `examples/` and `examples/starter.design.json`: Demo inputs/outputs.
- Tests live alongside code in `__tests__` folders (for example, `src/core/**/__tests__`).

## Build, Test, and Development Commands
- `npm run dev`: Starts both app and server via `concurrently`.
- `npm run dev:app`: Starts the Vite dev server only.
- `npm run dev:server`: Starts the `server/index.ts` watcher.
- `npm run build`: TypeScript project build + Vite production build.
- `npm run typecheck`: Strict TS project checks without emitting.
- `npm run lint`: ESLint across the repo with zero warnings allowed.
- `npm run test`: Runs Vitest once.
- `npm run test:watch`: Runs Vitest in watch mode.
- `npm run export`: Runs `scripts/export.mts` with `tsx`.

## Coding Style & Naming Conventions
- Language: TypeScript (`.ts` / `.tsx`), React 19, Vite 7.
- Indentation: 2 spaces; prefer clear, explicit code over cleverness.
- Components: PascalCase filenames (for example, `InspectorPanel.tsx`).
- Hooks/utilities: `useX.ts` / `xHelpers.ts`.
- Stores: Zustand stores end with `Store` (for example, `selectionStore.ts`).
- Styling: Tailwind utilities + CSS tokens in `src/styles/tokens.css`.
- Linting: ESLint is the source of truth. Run `npm run lint` before PRs.

## Testing Guidelines
- Framework: Vitest + Testing Library (`jsdom` environment).
- Location: Co-locate tests in `__tests__` directories.
- Naming: `*.test.ts` / `*.test.tsx`.
- Always run: `npm run typecheck && npm run test && npm run lint`.

## Commit & Pull Request Guidelines
- Follow the existing convention: `feat: ...`, `docs: ...`, `fix: ...`.
- Keep commits focused and scoped to a single change.
- PRs should include:
  - A short problem/solution summary.
  - Clear validation steps/commands.
  - Screenshots or recordings for UI changes.

## Architecture Notes (Quick Map)
- Core logic lives under `src/core/` (ops, model, history, collab).
- Feature surfaces live under `src/features/` (canvas, layers, inspector, comments).
- App layout lives under `src/app/layout/`.

---
> Source: [derrickchoi-openai/design-desk](https://github.com/derrickchoi-openai/design-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
