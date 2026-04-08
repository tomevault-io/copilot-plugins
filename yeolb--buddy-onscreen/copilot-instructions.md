## buddy-onscreen

> - `main/` holds Electron main-process entry (`main.ts`), IPC channels, and SQLite helpers.

# Repository Guidelines

## Project Structure & Module Organization
- `main/` holds Electron main-process entry (`main.ts`), IPC channels, and SQLite helpers.
- `renderer/` stores UI code: static HTML, TypeScript view logic, styles, and assets for the companion.
- `data/` retains seed content (compliments, badge tables) and migration scripts.
- `scripts/` aggregates automation such as packaging, schema generation, and smoke-test helpers.
- Keep feature experiments in `playgrounds/` so MVP code stays compact.

## Build, Test, and Development Commands
- `npm run dev` launches the Electron app with hot reload on main and renderer bundles.
- `npm run build` compiles TypeScript, packages assets, and emits a distributable bundle.
- `npm run lint` executes ESLint + Prettier for both processes; fix issues before committing.
- `npm run test` runs unit tests (Vitest) and lightweight IPC smoke checks.
- `npm run migrate` applies pending SQLite migrations to the local `buddy.db`.

## Coding Style & Naming Conventions
- TypeScript throughout; prefer explicit return types on exported functions.
- Use PascalCase for classes, camelCase for functions/variables, and kebab-case for filenames.
- Renderer templates live in `renderer/views/`; give DOM IDs descriptive prefixes (e.g., `timerStartButton`).
- Rely on ESLint (`@typescript-eslint`) and Prettier; do not disable rules without rationale inline.

## Testing Guidelines
- Place unit tests beside sources as `*.spec.ts`; mock Electron APIs with `@electron-mock/ipc`.
- Edge cases: empty achievement text, duplicate timers, and AI fallback messaging.
- Minimum expectation: each feature merges with passing unit tests plus a manual smoke run of the logging flow.

## Commit & Pull Request Guidelines
- Follow Conventional Commits (`feat:`, `fix:`, `chore:`) and keep messages under 72 characters.
- PR description should outline behavior changes, screenshots/gifs for UI tweaks, and list any new dependencies.
- Link related issues using `Closes #ID`; request review after lint/tests pass locally.

## Future Enhancements & Agents
- Treat voice capture, Live2D animation, and cloud sync as feature flags toggled in `main/config.ts`.
- Document new IPC channels in `docs/ipc.md`; provide mock responses for agent testing before hitting external APIs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yeolB)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yeolB)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
