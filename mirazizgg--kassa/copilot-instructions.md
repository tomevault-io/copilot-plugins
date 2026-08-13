## kassa

> Kassa is a POS system with a FastAPI backend and React frontend. Backend code is in `backend/`: `main.py` starts the application, `database.py` contains async SQLAlchemy models, `schemas.py` holds Pydantic DTOs, and `routers/` groups API endpoints by domain (for example, `pos.py`, `inventory.py`, and `crm.py`). Shared backend helpers live in `utils/`; the Telegram bot is `bot.py`.

# Repository Guidelines

## Project Structure & Module Organization

Kassa is a POS system with a FastAPI backend and React frontend. Backend code is in `backend/`: `main.py` starts the application, `database.py` contains async SQLAlchemy models, `schemas.py` holds Pydantic DTOs, and `routers/` groups API endpoints by domain (for example, `pos.py`, `inventory.py`, and `crm.py`). Shared backend helpers live in `utils/`; the Telegram bot is `bot.py`.

Frontend code is in `frontend/src/`. Put route-level screens in `pages/`, reusable components in `components/`, shadcn-style primitives in `components/ui/`, and HTTP/query setup in `api/`. Static files belong in `frontend/public/`. Do not commit local databases, `.env` files, or generated build output.

## Build, Test, and Development Commands

From the repository root, install backend dependencies with `pip install -r backend/requirements.txt`. Start the API with `cd backend; uvicorn main:app --reload --port 8000` (or `python main.py`).

For the UI, run `cd frontend; npm install`, then `npm run dev`. Use `npm run build` to produce a production bundle, `npm run preview` to inspect it, and `npm run lint` before submitting frontend changes. `./run_project.ps1` starts both services but terminates existing Python and Node processes; use it only when that is intended.

## Coding Style & Naming Conventions

Follow the style already present in the edited file. Python uses four-space indentation, `snake_case` functions/modules, `PascalCase` models and Pydantic schemas, type annotations, and async database access (`await db.execute(...)`). Add new endpoints to the appropriate domain router rather than `main.py`.

React uses ES modules, `PascalCase.jsx` for components and pages (for example, `SalesHistory.jsx`), camelCase variables/functions, and Tailwind utility classes. ESLint is configured for `.js` and `.jsx`; resolve lint errors instead of disabling rules.

## Testing Guidelines

There is no unified test runner or coverage threshold. Existing root-level checks are executable scripts such as `python test_features.py`, `python test_async_db.py`, and `python test_render_400.py`; some require a running API or configured database. For new behavior, add focused automated coverage where practical and manually verify affected API routes and UI flows.

## Commit & Pull Request Guidelines

Git history is unavailable in this checkout, so no repository-specific commit pattern can be confirmed. Use short imperative subjects, preferably scoped, such as `fix(pos): prevent sales without an open shift`. Keep commits focused. Pull requests should explain the user-facing change, list verification commands, link relevant issues, and include screenshots for visible frontend changes. Flag schema, environment-variable, or Telegram-bot configuration changes clearly.

## Security & Configuration

Copy example environment configuration rather than committing secrets. Keep JWT keys, bot tokens, and production database credentials out of source control. Treat database reset and shift-repair utilities as operational actions and run them only against the intended database.

---
> Source: [mirazizGG/Kassa](https://github.com/mirazizGG/Kassa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
