## chatcodex

> `backend/app/` contains the FastAPI gateway, MCP server, authentication, Codex App Server adapters, and tunnel managers. Tests live in `backend/tests/`: `test_contracts.py` covers protocol contracts, while `auth_http_integration.py` starts a real gateway. `frontend/src/` contains React widgets, the administration panel, bridge utilities, and shared controls. Widget entry points are in `frontend/widgets/`; builds go to `frontend/dist/`. Treat `native/` as downloaded runtimes and secrets, and `ref/` as reference material.

# Repository Guidelines

## Project Structure & Module Organization

`backend/app/` contains the FastAPI gateway, MCP server, authentication, Codex App Server adapters, and tunnel managers. Tests live in `backend/tests/`: `test_contracts.py` covers protocol contracts, while `auth_http_integration.py` starts a real gateway. `frontend/src/` contains React widgets, the administration panel, bridge utilities, and shared controls. Widget entry points are in `frontend/widgets/`; builds go to `frontend/dist/`. Treat `native/` as downloaded runtimes and secrets, and `ref/` as reference material.

## Build, Test, and Development Commands

- `cd backend && uv sync --locked` installs the reproducible Python environment from `uv.lock`.
- `cd backend && uv run python -m app.main` starts the gateway at `http://127.0.0.1:8000`.
- `cd backend && uv run python -m unittest discover -s tests -p "test_*.py"` runs backend contract tests.
- `cd backend && python tests/auth_http_integration.py` runs the real-process auth/OAuth smoke test.
- `cd frontend && npm ci` installs the locked frontend dependencies.
- `cd frontend && npm run dev` starts Vite; open `/preview.html` for widget inspection.
- `cd frontend && npm run build` produces standalone widget and panel HTML files.
- `cd frontend && npx tsc --noEmit` performs strict TypeScript checking.

## Coding Style & Naming Conventions

Use four spaces for Python and two for TypeScript/TSX. Python uses `snake_case` functions/modules, `PascalCase` classes, type hints, and short architectural docstrings. React components use `PascalCase`, hooks use `useXxx`, and widget entries use kebab case (for example, `start-codex.tsx`). Preserve double-quoted, semicolon-terminated TypeScript. No repository-wide formatter is configured; follow adjacent code.

## Testing Guidelines

Add regression tests to `test_contracts.py` for RPC shapes, authorization boundaries, settings, and MCP metadata. Name tests `test_<behavior>`. After cross-layer changes, run contract tests and the frontend build. For widgets, verify light/dark and inline/fullscreen layouts in `preview.html`.

## Commit & Pull Request Guidelines

This checkout contains no usable Git history, so use concise imperative commits such as `Fix OAuth resource metadata`. Keep unrelated backend, frontend, and runtime changes separate. PRs should explain behavior changes, list verification commands, link relevant issues, and include screenshots for panel or widget UI changes.

## Security & Configuration

Never commit generated access tokens, `native/secrets/`, database files, or tunnel credentials. Keep Web and MCP tokens distinct. Review changes to OAuth callbacks, public URLs, sandbox modes, approval policies, and MCP allowlists as security-sensitive.

---
> Source: [AeroidesLab/ChatCodex](https://github.com/AeroidesLab/ChatCodex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
