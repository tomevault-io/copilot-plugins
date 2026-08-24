## re-dollars-preact

> This repository is the next-generation Vite + Preact TypeScript frontend for the Re:Dollars userscript. Source code lives in `src/`. UI components are in `src/components/`, reusable hooks in `src/hooks/`, framework-light services in `src/services/`, signal-backed state in `src/stores/`, shared helpers in `src/utils/`, API clients in `src/utils/api/`, styles in `src/styles/index.css`, and shared types in `src/types/`. The userscript entry points are `src/main.tsx` and `src/App.tsx`. Build output is generated as `dist/userscript.user.js`; do not hand-edit it unless explicitly required. Tests sit next to the code they cover, using `*.test.ts`.

# Repository Guidelines

## Project Structure & Module Organization

This repository is the next-generation Vite + Preact TypeScript frontend for the Re:Dollars userscript. Source code lives in `src/`. UI components are in `src/components/`, reusable hooks in `src/hooks/`, framework-light services in `src/services/`, signal-backed state in `src/stores/`, shared helpers in `src/utils/`, API clients in `src/utils/api/`, styles in `src/styles/index.css`, and shared types in `src/types/`. The userscript entry points are `src/main.tsx` and `src/App.tsx`. Build output is generated as `dist/userscript.user.js`; do not hand-edit it unless explicitly required. Tests sit next to the code they cover, using `*.test.ts`.

## Build, Test, and Development Commands

- `npm install`: install dependencies from `package-lock.json`.
- `npm run dev`: start the Vite dev server for local development.
- `npm test`: run the full Vitest suite once.
- `npm test -- src/utils/bbcode.test.ts`: run a specific test file.
- `npm run typecheck`: run strict TypeScript checks without emitting files.
- `npm run build`: produce the distributable userscript in `dist/userscript.user.js`.
- `npm run preview`: preview the production build locally.

## Coding Style & Naming Conventions

Use TypeScript and Preact functional components. Keep indentation at 4 spaces, matching the existing code. Use `PascalCase` for components (`ChatInput.tsx`), `camelCase` for functions and variables, and `useXxx` for hooks. Prefer existing aliases such as `@/utils/...` and local patterns before introducing new abstractions. Keep comments short and only where they clarify non-obvious logic.

## Testing Guidelines

Vitest is the test framework. Add focused tests beside the implementation, for example `src/utils/mentions.test.ts` or `src/hooks/useMediaUpload.test.ts`. Prefer behavior-level assertions over implementation details. Run targeted tests while developing, then `npm test` before finishing. If a full-suite failure is unrelated, document the exact failing test and why it is out of scope.

## Commit & Pull Request Guidelines

Recent history uses concise conventional-style messages such as `feat: ...`, `fix(quote): ...`, and `refactor(SearchPanel): ...`. Follow that pattern with a clear scope when useful. PRs should include a short summary, test results, linked issue or context, and screenshots or screen recordings for visible UI changes. Mention any generated `dist/userscript.user.js` updates.

## Configuration & Security Notes

Public service URLs are defined in `src/utils/constants.ts`. Avoid committing secrets or local credentials. Upload behavior crosses the frontend, backend, and image host, so include endpoint URLs, HTTP status, and response body when reporting upload bugs.

This project may sit next to local credential JSON files during development. Treat every unrelated JSON file outside the package manifests and TypeScript config as a local secret.

---
> Source: [maho0x/re-dollars-preact](https://github.com/maho0x/re-dollars-preact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
