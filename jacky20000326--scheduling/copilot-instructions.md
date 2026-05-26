## scheduling

> - Root entry: `index.html` (Vite). Build config in `vite.config.ts`.

# Repository Guidelines

## Project Structure & Module Organization
- Root entry: `index.html` (Vite). Build config in `vite.config.ts`.
- App code lives in `src/`:
  - `src/App.tsx` – main UI component.
  - `src/main.tsx` – bootstraps React and mounts the app.
  - `src/index.css` – global styles.
- Build output: `dist/` (generated). Do not edit by hand.
- Docs: `README.md` describes usage at a glance.

## Build, Test, and Development Commands
- `npm run dev` – start Vite dev server with HMR.
- `npm run build` – create production build into `dist/`.
- `npm run preview` – serve the production build locally.
- Recommended Node: 18+ (required by Vite 5).

## Coding Style & Naming Conventions
- Language: React 18 with ES Modules.
- Indentation: 2 spaces; keep lines concise and focused.
- Components: PascalCase (e.g., `ShiftTable.tsx`, `ScheduleView.tsx`).
- Hooks/utilities: camelCase (e.g., `useShiftStore`, `formatDate`).
- File naming: `*.tsx` for React components; colocate small component styles; keep global rules in `src/index.css`.
- Imports: absolute within `src/` only if configured; otherwise use relative paths.
- No linters configured yet—match existing style and keep diffs minimal.

## Testing Guidelines
- Test tooling is not configured in this repo.
- If adding tests, prefer Vitest + React Testing Library.
  - Location: `src/__tests__/`
  - Naming: `*.test.tsx` (e.g., `App.test.tsx`).
  - Focus on rendering, props, and user flows.

## Commit & Pull Request Guidelines
- Use clear, scoped commits. Conventional Commits are encouraged:
  - Examples: `feat: add shift drag-and-drop`, `fix: correct date parsing`.
- PRs should include:
  - Purpose and scope; link related issue(s).
  - Screenshots/GIFs for UI changes.
  - Steps to validate locally (`npm run dev`/`npm run preview`).
  - Note any follow‑ups or tradeoffs.
- Do not commit changes in `dist/`; CI/release builds will regenerate it.

## Security & Configuration Tips
- Keep secrets out of the repo; use environment variables if introduced later.
- Avoid using `eval`/dangerous DOM APIs. Sanitize any dynamic HTML.
- Validate date/time inputs for scheduling to prevent edge‑case crashes.

---
> Source: [Jacky20000326/scheduling](https://github.com/Jacky20000326/scheduling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
