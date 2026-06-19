## mazes

> Keep this file under 60 lines. It's loaded every iteration.

# AGENTS.md - Operational Guide

Keep this file under 60 lines. It's loaded every iteration.

## Build Commands

```bash
npm run build          # TypeScript compile + Vite production build
npm run dev            # Vite dev server with HMR
```

## Test Commands

```bash
npm run test:run       # Run unit tests once (vitest)
npm run test:coverage  # Coverage report
npm run test:e2e       # Playwright end-to-end tests
```

## Lint & Format

```bash
npm run lint           # ESLint
npm run lint:fix       # ESLint with auto-fix
npm run format         # Prettier write
npm run format:check   # Prettier check
npm run typecheck      # tsc --noEmit
```

## Validation (run before committing)

```bash
npm run build && npm run test:run && npm run lint
```

## Tech Stack

- TypeScript, Vite, Three.js (3D), Canvas 2D API (2D views)
- Vanilla DOM for UI — no framework
- vitest for unit tests, Playwright for e2e
- ESLint + Prettier + Husky + lint-staged

## Project Notes

- Maze data model uses wall-per-cell (update both adjacent cells when removing a wall)
- Generators and solvers use JS generators (`function*`) for step-by-step animation
- Four view modes: top-down, isometric, first-person 3D, third-person 3D
- See plan.md for full architecture and specs/ for requirements

---
> Source: [ChrisBrooksbank/mazes](https://github.com/ChrisBrooksbank/mazes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
