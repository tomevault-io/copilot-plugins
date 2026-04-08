## sales-reporting-go

> - `frontend/`: Vite + React app. Entry in `src/main.jsx`, UI in `src/App.jsx`, styles in `src/styles.css`, config defaults in `src/config.js`, API helpers in `src/api/client.js`, and token helpers expected in `src/lib/token.js` (imported by `App.jsx`).

# Repository Guidelines

## Project Structure & Modules
- `frontend/`: Vite + React app. Entry in `src/main.jsx`, UI in `src/App.jsx`, styles in `src/styles.css`, config defaults in `src/config.js`, API helpers in `src/api/client.js`, and token helpers expected in `src/lib/token.js` (imported by `App.jsx`).
- `frontend/template.yaml` and `vite.config.js`: SAM + Vite build/deploy config for the static site.
- `backend/`: FastAPI Lambda. Core logic in `app/main.py`, dependencies in `requirements.txt`, SAM stack in `template.yaml`.
- Root scripts: `deploy-frontend.ps1`, `deploy-backend.ps1`, and docs in `README.md` and `DEPLOY.md`.
- Live deployment details are tracked in `DEPLOYMENT_STATUS.md`; reference it before changing stack names, regions, or secrets.

## Build, Test, and Development Commands
- Frontend: `cd frontend && npm install && npm run dev` (local dev server), `npm run build` (Vite prod build), `npm run preview` (serve built assets).
- Backend: `cd backend && python -m venv .venv && .\.venv\Scripts\Activate.ps1 && pip install -r requirements.txt` to bootstrap; `uvicorn app.main:app --reload` for local API, `sam build` then `sam deploy ...` for packaging.
- Deploy helpers: PowerShell scripts in the repo wrap the SAM deploy steps for each stack.

## Coding Style & Naming Conventions
- JavaScript/React: ES modules, functional components, hooks. Prefer named exports for utilities. Keep components in `PascalCase.jsx`, helpers in `camelCase.js`. Use Vite env vars (`import.meta.env`) with `VITE_` prefix.
- Python: FastAPI style with type hints and dependency injection. Functions and variables in `snake_case`; constants in `UPPER_SNAKE_CASE`. Avoid global state beyond cached JWKS.
- Formatting: Follow existing spacing/quote style (double quotes in JS, 4-space indents in Python). Run Prettier/Black if added later; none enforced yet.

## Testing Guidelines
- No automated tests ship yet. For backend changes, add lightweight FastAPI route tests (e.g., pytest + httpx) and verify token failure/success cases. For frontend, add React/Vite tests (Vitest/RTL) around token parsing and API client behaviors.
- Manual checks: run `npm run dev` with a sample `?sso_token=` query, and hit `/health`, `/greeting`, `/me` locally via `uvicorn`.

## Commit & Pull Request Guidelines
- Commits: short imperative titles (“Add token decode helpers”). Keep related changes together; avoid large mixed commits.
- PRs: include summary, testing notes (commands run), and any screenshots of UI changes. Link related issues/tickets. Call out env var or infra parameter changes (`VITE_API_BASE_URL`, `CUSTOM_ISSUER`, `ALLOWED_ORIGINS`, etc.).

## Security & Configuration
- Never commit real tokens or AWS secrets. Use `.env` for frontend overrides and stack parameters for backend secrets. Ensure `AllowedFrameAncestors`/CORS settings remain restricted to the Doc Viewer origin.
- Token handling: keep `getTokenFromLocation`, `decodeClaims`, and auth header construction in `src/lib/token.js` pure and side-effect free; all backend calls must include the bearer token.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nliening)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nliening)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
