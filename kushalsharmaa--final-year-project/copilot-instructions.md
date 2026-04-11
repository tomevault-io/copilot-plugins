## final-year-project

> - Root splits into `backend/` (Flask API for speech assessment) and `frontend/` (Create React App with Tailwind). `firestore.rules` holds Firebase rules.

# Repository Guidelines

## Project Structure & Module Organization
- Root splits into `backend/` (Flask API for speech assessment) and `frontend/` (Create React App with Tailwind). `firestore.rules` holds Firebase rules.
- Backend: `app.py` boots Flask, health checks, and `/api/assess`; `assessment.py` scores transcripts; optional `voice_security.py` blueprint; assets in `pretrained_models/`, `storage/`, and `voice_data/`. Env is read from `backend/.env` plus `GOOGLE_APPLICATION_CREDENTIALS`.
- Frontend: `src/` contains `components/`, `pages/`, `features/`, `lib/`, and `types/`; entry files `index.js` and `App.js`. Static assets live in `public/`.

## Setup, Build, and Test
- Backend (Python 3.10+): `cd backend && python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt`.
- Run API: `cd backend && source .venv/bin/activate && python app.py` (port 5050). Set `WHISPER_MODEL`/`WHISPER_COMPUTE` and `GOOGLE_APPLICATION_CREDENTIALS` in `.env`. Smoke test: `curl localhost:5050/api/health`.
- Frontend: `cd frontend && npm install`; dev server `npm start`; production build `npm run build`; tests `npm test` (Jest + RTL watch mode).

## Coding Style & Naming
- React: functional components + hooks; Tailwind utilities; PascalCase components, camelCase props/helpers; colocate UI logic under `src/components` or `src/features`. React Scripts’ ESLint is enabled—fix warnings before commit.
- Python: PEP8 with 4 spaces and snake_case. Keep handlers small and log via `logging`; put pure logic in helpers like `assessment.py`. Add type hints when touching shared utilities.
- Tests as `*.test.js`; backend helpers as `lower_snake.py`.

## Testing Expectations
- Frontend: cover new UI states with RTL (happy + error paths); mock network calls with `jest.fn` or `msw`. Snapshot only stable components.
- Backend: no suite yet—add `pytest`-style unit tests for new logic (scoring, parsers). Verify endpoints manually with curl, including `Authorization` when hitting Firebase-backed routes.

## Commit & Pull Request Guidelines
- Commits: imperative, concise subjects (e.g., `Add whisper health check`, `Fix assess payload validation`); keep scopes tight.
- PRs: include a short summary, linked issue, screenshots/clips for UI changes, and a test plan (`npm test`, manual endpoint checks). Note any config/env needs for reviewers.

## Security & Configuration Tips
- Do not commit secrets; use `backend/.env` and point to service accounts with `GOOGLE_APPLICATION_CREDENTIALS`. Avoid embedding keys in React code.
- Large models/audio belong in `backend/pretrained_models` or `voice_data`; discuss before adding heavy files. CORS is open to localhost—update carefully when adding domains or endpoints.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kushalsharmaa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kushalsharmaa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
