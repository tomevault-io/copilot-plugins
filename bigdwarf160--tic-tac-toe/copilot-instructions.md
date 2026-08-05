## tic-tac-toe

> This repository is a React + Vite TypeScript tic-tac-toe app. Source code lives in

# Repository Guidelines

## Project Structure & Module Organization

This repository is a React + Vite TypeScript tic-tac-toe app. Source code lives in
`src/`. The main React entrypoint is `src/main.tsx`, the primary UI component is
`src/App.tsx`, reusable game rules are in `src/game.ts`, and unit tests sit next
to the code as `src/game.test.ts`. Global styling is in `src/styles.css`.
Generated output goes to `dist/`; dependencies are installed in `node_modules/`.
Do not edit generated folders directly.

## Build, Test, and Development Commands

Use `npm.cmd` on Windows PowerShell if `npm` is blocked by script policy.

- `npm.cmd install`: install dependencies from `package-lock.json`.
- `npm.cmd run dev -- --port 5173`: start the Vite dev server locally.
- `npm.cmd test`: run Vitest tests once.
- `npm.cmd run build`: type-check with TypeScript and create a production build.

## Coding Style & Naming Conventions

Use TypeScript with strict types. Prefer small pure functions for game logic and
keep React components focused on rendering and user interaction. Use two-space
indentation, double quotes, semicolons, and trailing commas where the existing
code uses them. Name React components in `PascalCase`, functions and variables
in `camelCase`, and shared types in `PascalCase` (`Player`, `Board`, `GameResult`).

## Testing Guidelines

Tests use Vitest. Place tests beside the module they cover with the
`*.test.ts` naming pattern. Cover game-rule changes with pure unit tests before
or alongside UI changes. At minimum, verify win detection, draw detection,
unfinished games, invalid moves, and turn changes when modifying game behavior.
Run `npm.cmd test` before submitting changes.

## Commit & Pull Request Guidelines

Git history is not available in this checkout, so use concise imperative commit
messages such as `Add score reset control` or `Fix draw detection`. Keep each
commit focused on one change. Pull requests should include a short summary,
testing notes, and screenshots or a short screen recording for visible UI
changes. Link related issues when available and call out any behavior changes
that affect gameplay.

## Security & Configuration Tips

This app has no backend or secrets. Keep configuration in source-controlled
Vite/TypeScript files unless a future feature explicitly requires environment
variables. Do not commit local logs, generated build artifacts, or dependency
folders.

---
> Source: [bigdwarf160/tic-tac-toe](https://github.com/bigdwarf160/tic-tac-toe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
