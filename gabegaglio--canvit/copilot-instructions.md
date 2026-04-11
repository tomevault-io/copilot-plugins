## canvit

> Canvit is a Vite + React canvas editor. Core entry (`src/Canvas.tsx`) orchestrates feature folders under `src/canvas/` (`components/`, `hooks/`, `menus/`, `utils/`). Reusable shadcn-inspired primitives live in `src/components/ui/`; extend them before introducing new atoms. Tailwind entry points (`src/App.css`, `src/index.css`) hold the glassmorphic tokens—tune them there, not ad hoc. Shared helpers belong in `src/lib/` or `src/utils/`, while authentication stays in `src/Auth.tsx`. Static assets live in `public/`; builds emit to `dist/`.

# Repository Guidelines

## Project Structure & Module Organization
Canvit is a Vite + React canvas editor. Core entry (`src/Canvas.tsx`) orchestrates feature folders under `src/canvas/` (`components/`, `hooks/`, `menus/`, `utils/`). Reusable shadcn-inspired primitives live in `src/components/ui/`; extend them before introducing new atoms. Tailwind entry points (`src/App.css`, `src/index.css`) hold the glassmorphic tokens—tune them there, not ad hoc. Shared helpers belong in `src/lib/` or `src/utils/`, while authentication stays in `src/Auth.tsx`. Static assets live in `public/`; builds emit to `dist/`.

## Build, Test, and Development Commands
Install deps via `npm install`. `npm run dev` launches the hot-reloading canvas workspace (`--host` for device testing). `npm run build` runs `tsc -b` plus Vite bundling; execute it before every PR. `npm run preview` serves the production bundle. `npm run lint` must pass—fix warnings rather than silencing them.

## UI Style & Component Patterns
Match the glassmorphic look from the theme overhaul: translucent layers, blurred panels, high-contrast text. Compose UIs with Tailwind utilities; encapsulate recurring layouts inside `src/components/ui/` components patterned after shadcn. When updating canvas overlays (e.g., `SnapGuide` or `ToolBarButtons`), keep DOM depth shallow and reuse motion tokens to avoid frame drops.

## Coding Style & Naming Conventions
Write TypeScript function components with hooks. Maintain 2-space indentation, trailing commas, and single quotes. Avoid useEffect where possible/ Name components `PascalCase`, hooks `useCamelCase`, utilities `camelCase`, constants `SCREAMING_CASE`. Use `clsx` for conditional classes; hoist shared Tailwind macros into helpers on third reuse. Adjust lint behavior in `eslint.config.js` only with a comment and TODO.

## Performance & Latency Requirements
Deliver clean, efficient code that keeps latency low at all costs while staying fully functional. Memoize expensive selectors, keep pointer handlers pure, and debounce network calls outside render paths. Profile new canvas interactions with React DevTools; if frame time exceeds 16 ms, document mitigation or follow-up work before merge.

## Testing Guidelines
Adopt Vitest + Testing Library, storing specs in `src/__tests__/` or beside features (`*.test.tsx`). Cover canvas mutations, undo/redo, and data transforms. Until coverage grows, list manual QA steps (e.g., “drag note with snap-on”) in PRs.

## Commit & Pull Request Guidelines
Prefer Conventional Commit subjects (`feat: add glass blur`). Keep subjects ≤72 chars and group related code per commit. Every PR needs a bullet summary, UI screenshots or a short Loom, commands executed (`npm run lint`, `npm run build`), linked issues, and any required env or flag toggles reviewers need.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gabegaglio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gabegaglio)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
