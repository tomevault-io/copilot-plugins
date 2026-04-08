## codex-sdk-ts-sample

> - `backend/`: Express + TypeScript API that wraps the Codex CLI; entry point `backend/src/index.ts`; compiled output in `backend/dist/`.

# Repository Guidelines

## Project Structure & Module Organization
- `backend/`: Express + TypeScript API that wraps the Codex CLI; entry point `backend/src/index.ts`; compiled output in `backend/dist/`.
- `frontend/`: React + Vite client with main bootstrap in `frontend/src/main.tsx` and UI logic in `frontend/src/App.tsx`; static assets live under `frontend/public/`, and production bundles land in `frontend/dist/`.
- Keep environment-specific configuration outside the repo (`.env` or shell exports) and document any new env vars in PRs.

## Build, Test, and Development Commands
- `cd backend && npm install`: install backend deps; `npm run dev`: run `tsx` watcher on http://localhost:4000; `npm run build`: type-check and emit `dist/`; `npm start`: serve the built bundle.
- `cd frontend && npm install`: install UI deps; `npm run dev`: start Vite dev server with proxy to backend; `npm run build`: run `tsc` then bundle; `VITE_API_BASE_URL=http://localhost:4000/api npm run preview`: preview production bundle.
- Export Codex credentials (`AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_KEY`, `AZURE_OPENAI_DEPLOYMENT`) before launching the backend; override `CODEX_WORKDIR` when pointing Codex to another directory.

## Coding Style & Naming Conventions
- TypeScript strict mode is enabled; prefer explicit types at module boundaries and validate request payloads via `zod` as in `backend/src/index.ts`.
- Use 2-space indentation, camelCase for variables/functions, PascalCase for React components, and descriptive file names (`ThreadList.tsx`, `useCodexClient.ts` if added).
- Run `npm run build` in both packages prior to PRs; match existing formatting until Prettier/ESLint configs are introduced.

## Testing Guidelines
- Automated tests are not yet configured; add future backend tests alongside code (`backend/src/__tests__/api.spec.ts`) and frontend tests under `frontend/src/__tests__/`.
- Fallback validation: hit `POST /api/run` with `curl` or REST clients, and walk through the UI flow to confirm prompt/response loops; capture manual steps in PR descriptions.

## Commit & Pull Request Guidelines
- Follow Conventional Commits, scoping messages where helpful (e.g., `feat(frontend): persist thread history`).
- Each PR should explain motivation, list manual or automated test results, call out environment variable changes, and include UI screenshots/log snippets when behavior shifts.
- Keep changes focused; update docs or configuration examples that drift alongside code updates.

## Security & Configuration Tips
- Never commit secrets; rely on `.env.local` or shell exports and document required variables.
- Limit backend access by using the minimal `CODEX_WORKDIR`; ensure dependencies are reviewed for security before upgrades.
- Remove temporary logging or debugging code before merging to avoid leaking prompt data.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nogataka)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nogataka)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
