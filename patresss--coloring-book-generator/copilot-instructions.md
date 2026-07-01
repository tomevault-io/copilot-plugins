## coloring-book-generator

> - Root: `.env.example`, planning docs (`plan.md`, `implementation_plan.md`).

# Repository Guidelines

## Project Structure & Modules
- Root: `.env.example`, planning docs (`plan.md`, `implementation_plan.md`).
- `backend/`: TypeScript Express API. Key folders: `src/routes` (REST endpoints), `src/services` (OpenAI, storage, printer), `src/utils`, `src/constants`, `data/` (generated sessions, images).
- `frontend/`: React + Vite. Key folders: `src/api` (HTTP client), `src/components`, `src/App.tsx`.
- Static files: served from backend at `/files/:id/...`.

## Build, Test, and Development
- Backend:
  - `cd backend && npm i`
  - `npm run dev`: start API with hot-reload on `:33000`.
  - `npm run build`: type-check/compile to `dist`.
  - `npm start`: run compiled server.
- Frontend:
  - `cd frontend && npm i`
  - `npm run dev`: Vite dev server on `:35173` with proxy to `:33000` for `/api` and `/files`.
  - `npm run build` / `npm run preview`: production build and preview.

## Coding Style & Naming
- TypeScript strict mode across repo; ESM modules.
- Indentation: 2 spaces; include semicolons.
- Backend files: lowercase names (e.g., `openai.ts`, `printer.ts`).
- React components: `PascalCase.tsx` (e.g., `App.tsx`); utilities and API clients: lowercase or camelCase (e.g., `api/client.ts`).
- Prefer named exports; keep modules focused.

## Testing Guidelines
- No test harness is configured yet. If adding tests:
  - Frontend: Vitest + React Testing Library, files `*.test.tsx` colocated with source.
  - Backend: Vitest/Jest + supertest, files `*.test.ts` under `src/`.
  - Aim for coverage on routes and critical services (OpenAI integration can be mocked).

## Commit & PR Guidelines
- Commits: imperative, concise, scoped. Examples: `Fix printer URI`, `Add history delete route`, `Refactor storage paths`.
- Pull Requests:
  - Clear description, rationale, and linked issue (if any).
  - Steps to test locally; include screenshots/GIFs for UI changes.
  - Note environment needs and breaking changes.

---
> Source: [Patresss/coloring-book-generator](https://github.com/Patresss/coloring-book-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
