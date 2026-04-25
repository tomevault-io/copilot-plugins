## nochatbot-data-analyze-helper

> - `backend/`: FastAPI API and analysis pipeline (`main.py`, `analyzer.py`, `llm.py`, `db.py`, `nl2sql.py`, `e2b_runner.py`).

# Repository Guidelines

## Project Structure & Module Organization
- `backend/`: FastAPI API and analysis pipeline (`main.py`, `analyzer.py`, `llm.py`, `db.py`, `nl2sql.py`, `e2b_runner.py`).
- `frontend/`: React 19 + TypeScript app (Vite). UI components are in `frontend/src/components/`; schema/catalog setup is in `frontend/src/lib/` and `frontend/src/components/registry.tsx`.
- `sample_data/`: local datasets for upload smoke tests.
- `resource/`: demo assets (videos/images).
- `.github/workflows/pylint.yml`: Python lint CI workflow.

## Build, Test, and Development Commands
- Backend setup: `cd backend && pip install -r requirements.txt`
- Run backend: `uvicorn main:app --reload --port 8000` (serves `http://localhost:8000`)
- Frontend setup: `cd frontend && npm install`
- Run frontend: `npm run dev` (serves `http://localhost:5173`)
- Lint frontend: `npm run lint` (ESLint for `ts/tsx`)
- Build frontend: `npm run build` (TypeScript build + Vite production bundle)
- Preview build: `npm run preview`

## Coding Style & Naming Conventions
- Python: 4-space indentation, `snake_case` for functions/variables, small focused functions.
- TypeScript/React: `PascalCase` for components (for example `StatCard.tsx`), `camelCase` for helpers/hooks.
- Keep component props aligned with the JSON render catalog contracts (`BarChart`, `LineChart`, `PieChart`, etc.).
- Keep imports tidy and remove unused variables before pushing.

## Testing Guidelines
- No full unit-test suite is committed yet.
- Minimum pre-PR checks:
  - `cd frontend && npm run lint`
  - `cd frontend && npm run build`
  - Manual smoke test: upload at least one file from `sample_data/` and verify dashboard rendering.
- For non-trivial backend parsing/query changes, add targeted automated tests where possible.

## Commit & Pull Request Guidelines
- Existing history is release-oriented (for example: `version 2.1.0 support personal database analyze`).
- Prefer concise, imperative commit subjects; include scope when useful (for example: `backend: tighten SQL guard`).
- PRs should include:
  - What changed and why
  - Verification steps
  - UI screenshots/GIFs for frontend changes
  - Linked issue/task reference

## Security & Configuration Tips
- Store secrets in `backend/.env` (`LLM_API_KEY`, `LLM_BASE_URL`, `LLM_MODEL`, `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`).
- Never commit credentials.
- Keep generated metadata such as `backend/db_meta.json` out of version control.

---
> Source: [21pounder/nochatbot-data-analyze-helper](https://github.com/21pounder/nochatbot-data-analyze-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
