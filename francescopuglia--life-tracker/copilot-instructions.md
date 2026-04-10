## life-tracker

> You are auditing and stabilizing this Next.js 15 + TypeScript project.

# Repository Guidelines (for Codex)

## Mission
You are auditing and stabilizing this Next.js 15 + TypeScript project.
Goal: get a clean production build + fix known UI bugs (Time Planner / Goals / Projects) with minimal, verified diffs.

## Project Structure & Module Organization
- Next.js app lives in `src/`; main route: `src/app/page.tsx`.
- UI components: `src/components/`
- Domain logic (auth/db/audio): `src/lib/`
- Firebase config: `src/config/`
- Utilities: `src/utils/`
- Styles: `src/styles/` + `src/app/globals.css`
- Types: `src/types/`
- Static assets: `public/`
- Build output: `out/` (do not edit manually)

## Working Rules (IMPORTANT)
1) Phase 1 = Audit (Read Only): map the codebase folder-by-folder, identify issues, propose plan.
2) Phase 2 = Fixes: apply ONE fix at a time, then validate (build/lint/tests) before moving on.
3) Prefer minimal diffs. Avoid refactors unless necessary to fix the bug.
4) After each fix: run `/review` in Codex and rerun the failing command(s).
5) Do not change product behavior unless explicitly required by a bug fix.

## Known P0 Build Breaker (must fix first)
- `npm run build` fails:
  `./src/components/OKRManager.tsx:906:8`
  TypeScript error: "Block-scoped variable 'KeyResultEditModal' used before its declaration."
  JSX uses `<KeyResultEditModal />` around that line.
  Fix requirement: `npm run build` must pass.

## Known Functional Bugs (must reproduce + fix)
### Time Planner
- Clicking "Add Block" does not add/show any Time Block.
- Expected: a new block appears immediately and persists (depending on storage layer).

### Goals / Projects
- Creating a Goal or a Project does not add it (no visible update / no persistence).

## Definition of Done
- `npm run build` succeeds.
- Time Planner "Add Block" works (UI updates + data persists consistently).
- Goals & Projects creation works (UI updates + data persists consistently).
- No new TypeScript errors introduced; lint passes if available.
- Changes are small and well explained.

## Commands (use in this order; stop at first failure and diagnose)
- Install: `npm install`
- Lint: `npm run lint` (if available)
- Tests: `npm test` (if available)
- Build: `npm run build`
- Dev: `npm run dev` and reproduce UI bugs (Time Planner / Goals / Projects)

## Debugging Approach
- When a UI action does nothing, trace:
  1) UI handler/onClick wiring
  2) state update path (React state/store)
  3) persistence (localStorage/Firebase/IndexedDB/etc.)
  4) rendering conditions (filters, date ranges, selected view)
  5) errors in console/logs and failed network calls (if any)

## Coding Style & Naming
- TypeScript + React hooks; follow existing patterns.
- 2-space indentation, single quotes, trailing commas.
- PascalCase for components, camelCase for functions/vars, UPPER_SNAKE_CASE constants.
- Prefer `@/` absolute imports.

## Security & Configuration
- Firebase config in `src/config/firebaseConfig.ts`.
- Do not commit secrets; prefer env vars for any new sensitive config.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FrancescoPuglia)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/FrancescoPuglia)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
