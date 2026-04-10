## klausweigele-ai-site

> The repo is a two-app monorepo: `frontend/` hosts the Next.js 14 marketing site (App Router, TypeScript) with UI components under `frontend/app/components` and API routes in `frontend/app/api/contact`. `backend/` contains the FastAPI service; domain logic lives in `backend/app` (config in `core/`, HTTP routes in `api/routes.py`, background helpers in `services/`), and tests reside in `backend/tests`. Shared assets enter the respective app directories; avoid adding cross-app code outside these folders.

# Repository Guidelines

## Project Structure & Module Organization
The repo is a two-app monorepo: `frontend/` hosts the Next.js 14 marketing site (App Router, TypeScript) with UI components under `frontend/app/components` and API routes in `frontend/app/api/contact`. `backend/` contains the FastAPI service; domain logic lives in `backend/app` (config in `core/`, HTTP routes in `api/routes.py`, background helpers in `services/`), and tests reside in `backend/tests`. Shared assets enter the respective app directories; avoid adding cross-app code outside these folders.

## Build, Test, and Development Commands
- `cd frontend && npm install && npm run dev` – launch the Next.js dev server on http://localhost:3000.
- `cd frontend && npm run lint` / `npm run type-check` – enforce ESLint rules and TypeScript constraints before committing.
- `cd backend && python3 -m venv .venv && source .venv/bin/activate && pip install -e .[dev]` – create the local environment with dev extras.
- `cd backend && uvicorn app.main:create_app --reload` – run the API on http://localhost:8000.
- `cd backend && pytest` – execute backend tests (HTTP client coverage in `tests/test_api.py`).
- `cd backend && ruff check app` – optional static analysis aligned with the dev dependency set.

## Coding Style & Naming Conventions
Follow TypeScript strictness and functional React patterns; use PascalCase for components (`ContactForm.tsx`), kebab-case for route folders, and 2-space indentation matching existing files. Keep CSS colocated via Next conventions. Python modules use 4-space indentation, full type annotations, and FastAPI dependency injection; extend `schemas.py` for Pydantic models and keep route logic thin by delegating to `services/`.

## Testing Guidelines
Extend `backend/tests` with descriptive `test_<feature>.py` modules, leveraging `pytest` fixtures and `httpx.AsyncClient` for FastAPI. Aim to cover new endpoints and service branches; target ~80% statement coverage before merging. Frontend tests are not yet scaffolded—add Vitest or React Testing Library cases under `frontend/__tests__/` mirroring the component path, and wire them into `package.json` once introduced.

## Commit & Pull Request Guidelines
Use short, imperative commit subjects with optional scope, mirroring `Initial commit: Next.js frontend and FastAPI backend`. Each PR should summarise user-facing impact, reference issue IDs, list test commands executed, and attach screenshots for UI updates. Keep diffs focused per feature or bugfix; rebase on `main` before requesting review.

## Environment & Configuration Tips
Work in zsh or source `~/.nvm/nvm.sh` so Node 18.20.4 is active; activate the Python venv for backend tasks. Frontend expects `.env.local` with `NEXT_PUBLIC_BACKEND_URL`, while backend reads `.env` (see `backend/.env.example`); avoid committing secrets by using the provided templates.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KlausWeigele)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KlausWeigele)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
