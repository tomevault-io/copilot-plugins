## glumoo-rednote

> - Backend: Flask app factory in `backend/app.py`; domain routes live in `backend/routes` (`outline`, `content`, `image`, `history`, `config`) calling logic in `backend/services` and provider adapters in `backend/generators`.

# Repository Guidelines

## Project Structure & Module Organization
- Backend: Flask app factory in `backend/app.py`; domain routes live in `backend/routes` (`outline`, `content`, `image`, `history`, `config`) calling logic in `backend/services` and provider adapters in `backend/generators`.
- Shared config in `backend/config.py`; YAML inputs `text_providers.yaml` and `image_providers.yaml`; runtime artifacts land in `data/history` and `data/output`.
- Frontend: Vite + Vue 3 app in `frontend/` with API helpers in `src/api`, state in `src/stores`, composables in `src/composables`, and UI pieces in `src/components`; assets reside in `public/` and `src/assets/`.
- Deployment helpers: Dockerfile, `docker-compose.yml`, and cross-platform launch scripts (`start.sh`, `scripts/start-*.sh|.command|.bat`).

## Build, Test, and Development Commands
- Install backend deps: `uv sync` (Python 3.11+).
- Run backend (dev): `uv run python -m backend.app` (listens on 12398).
- Install frontend deps: `cd frontend && pnpm install` (Node 18+).
- Run frontend: `pnpm dev` (5173); build for prod: `pnpm build`.
- Docker quickstart: `docker-compose up -d` or `./start.sh` for OS-aware boot.

## Coding Style & Naming Conventions
- Python: follow PEP 8, 4-space indents, snake_case for functions/variables, PascalCase for classes; prefer type hints when practical and reuse the existing logging patterns for request tracing.
- Vue/TypeScript: PascalCase component files, camelCase props/data, `use*` naming for composables and Pinia stores; keep API calls centralized in `src/api` rather than inside view components.
- YAML configs: keep readable spacing (no tabs); never commit real API keys—use env vars like `GOOGLE_API_KEY`/`GEMINI_API_KEY` or local `text_providers.yaml`/`image_providers.yaml`.

## Testing Guidelines
- Pytest scaffolding lives in `tests/` with fixtures in `tests/conftest.py`; add new `test_*.py` files to cover routes/services.
- Run tests with `uv run pytest`; if missing locally, install via `uv add --dev pytest`.
- Favor fast, deterministic tests: use temp dirs instead of repo paths and stub external provider calls.

## Commit & Pull Request Guidelines
- Recent history uses conventional prefixes (`fix:`); keep short, lower-case summaries (e.g., `feat: add outline retry`, `chore: update docker config`) and prefer the same tone (English or concise Chinese) project-wide.
- For PRs, include scope/intent, config changes (text/image providers, env vars), screenshots for UI tweaks, and local test commands you ran. Link issues when relevant and flag deployment impacts.

---
> Source: [Yeadon8888/Glumoo_rednote](https://github.com/Yeadon8888/Glumoo_rednote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
