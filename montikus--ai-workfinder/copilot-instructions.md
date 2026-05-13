## ai-workfinder

> `backend/app` contains the FastAPI service. Keep HTTP routes in `backend/app/api/routes`, shared dependencies in `backend/app/api/deps.py`, database code in `backend/app/db`, business logic in `backend/app/services`, and Pydantic schemas in `backend/app/schemas`. Repository-layer Mongo helpers live in `backend/app/repositories`. Scraping and apply helpers are in `backend/app/tools`. Uploaded files land in `backend/uploads` and should not be hand-edited.

# Repository Guidelines

## Project Structure & Module Organization
`backend/app` contains the FastAPI service. Keep HTTP routes in `backend/app/api/routes`, shared dependencies in `backend/app/api/deps.py`, database code in `backend/app/db`, business logic in `backend/app/services`, and Pydantic schemas in `backend/app/schemas`. Repository-layer Mongo helpers live in `backend/app/repositories`. Scraping and apply helpers are in `backend/app/tools`. Uploaded files land in `backend/uploads` and should not be hand-edited.

`job-agent-frontend/src` contains the Vite + React UI. Put page-level screens in `src/pages`, reusable layout and widgets in `src/components`, API clients in `src/api`, auth/i18n state in `src/context`, and route guards in `src/router`. Static assets belong in `job-agent-frontend/public` or `src/assets`.

## Build, Test, and Development Commands
Backend setup:

- `pip install -r backend/requirements.txt` installs the FastAPI, Mongo, and AI dependencies.
- `cd backend && uvicorn app.main:app --reload --port 8001` starts the API locally.
- `docker run -d --name mongodb -p 27017:27017 mongo:6` starts the MongoDB instance used by local development.

Frontend setup:

- `cd job-agent-frontend && npm install` installs the Vite/React toolchain.
- `cd job-agent-frontend && npm run dev` starts the frontend dev server on Vite’s default port.
- `cd job-agent-frontend && npm run build` creates a production bundle.
- `cd job-agent-frontend && npm run lint` runs ESLint.

## Coding Style & Naming Conventions
Follow the existing style in each half of the repo: Python uses 4-space indentation, snake_case modules, and typed FastAPI/Pydantic functions; JavaScript/JSX uses 2-space indentation, semicolons, and PascalCase component files such as `DashboardPage.jsx` or `MainLayout.jsx`. Keep frontend API helpers lowercase (`auth.js`, `jobs.js`). Reuse the current domain vocabulary where it already appears, including Polish backend identifiers.

## Testing Guidelines
There is no committed end-to-end test suite or enforced coverage gate yet. Before opening a PR, run `npm run lint`, `npm run build`, and smoke-test the backend with `GET /health` plus the route you changed. If you add backend tests, prefer `backend/tests/test_<feature>.py` and keep them isolated from live services where possible.

## Commit & Pull Request Guidelines
Recent history uses short, lowercase commit summaries such as `translation button` and `clean up`. Keep commits brief, specific, and focused on one change. Pull requests should describe the user-visible impact, note backend/frontend touch points, link the issue if one exists, and include screenshots for UI changes. Do not commit secrets; configure `MONGODB_URI`, `MONGODB_DB_NAME`, `JWT_SECRET_KEY`, `JWT_ALGORITHM`, and `JWT_EXPIRE_MINUTES` through environment variables.

---
> Source: [montikus/ai_workfinder](https://github.com/montikus/ai_workfinder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
