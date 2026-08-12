## videoroll

> `src/videoroll/` is the Python 3.12 backend. `apps/monolith` exposes `/api`; `apps/subtitle_service` owns ASR, translation, render queues, and the RAG Agent runtime; `apps/youtube_ingest` and `apps/bilibili_publisher` handle ingest and publishing. Shared config, DB, storage, AI, and utilities live in `config.py`, `db/`, `storage/`, `ai/`, and `utils/`. `src/web/` is React 18 + Vite + Tailwind, with pages, helpers, and tests in `src/web/src/pages/`. Backend tests are `tests/test_*.py`. Docs live in `docs/`; local state in `data/`. Avoid casual edits to vendored `biliup-master/` and `bilibili-API-collect-main/`.

# Repository Guidelines

## Project Structure & Module Organization
`src/videoroll/` is the Python 3.12 backend. `apps/monolith` exposes `/api`; `apps/subtitle_service` owns ASR, translation, render queues, and the RAG Agent runtime; `apps/youtube_ingest` and `apps/bilibili_publisher` handle ingest and publishing. Shared config, DB, storage, AI, and utilities live in `config.py`, `db/`, `storage/`, `ai/`, and `utils/`. `src/web/` is React 18 + Vite + Tailwind, with pages, helpers, and tests in `src/web/src/pages/`. Backend tests are `tests/test_*.py`. Docs live in `docs/`; local state in `data/`. Avoid casual edits to vendored `biliup-master/` and `bilibili-API-collect-main/`.

## Build, Test, and Development Commands
Compose starts `app`, `web`, Redis, and MinIO. PostgreSQL 16+ is external; configure `DATABASE_URL`.

- `./scripts/dev_up.sh`: create `.env` if missing, build, and start locally.
- `./scripts/dev_down.sh`, `./scripts/dev_logs.sh`, `./scripts/dev_health.sh`: stop, inspect logs, or check health.
- `./scripts/dev_web.sh`: run only Vite on port `3000`.
- `python -m pytest tests/`: run backend tests.
- `cd src/web && npm run lint && npm run test && npm run build`: lint, test, and build the frontend.
- `./scripts/smoke_local.sh [video.mp4]`: run an upload/subtitle smoke flow.
- `./scripts/build_export_prod.sh`: build and export Docker images.

## Coding Style & Naming Conventions
Follow the existing style; do not introduce a new formatter. Python uses 4-space indentation, type hints, `snake_case` for modules/functions, and `PascalCase` for classes and Pydantic models. Keep service code in the matching `apps/*` package and prefer structured DB/API helpers. Frontend components use `PascalCase`, helpers use names like `videosPage.helpers.ts`, and tests use `*.test.ts`. Run ESLint for UI changes.

## Testing Guidelines
Backend tests use `pytest`, with `unittest.TestCase` and plain pytest functions. Add coverage in `tests/test_<feature>.py`, especially for queues, RAG retrieval/agent budgets, publishing retries, and parsing. Mock network, `yt-dlp`, LLMs, and external APIs. Keep frontend tests colocated. Targeted examples: `python -m pytest tests/test_translation_rag.py` or `cd src/web && npm run test`.

## Commit & Pull Request Guidelines
Recent history favors short imperative subjects with Conventional Commit prefixes, for example `feat: add tool-driven RAG agents` or `fix: harden publish retry and proxy checks`. Keep commits focused. PRs should include a summary, linked issue when applicable, config/schema notes, UI screenshots, and verification commands.

## Security & Configuration Tips
Start from `.env.example`. Never commit real cookies, API keys, generated model data, or `data/secrets/fernet.key`; losing that key makes encrypted DB settings unreadable. RAG vector search may require pgvector. Treat downloader, auth, publishing, LLM tools, URL fetching, and secret storage as security-sensitive. Document new env vars and keep tests offline with bounded tool/fetch budgets.

---
> Source: [kelriclink/videoroll](https://github.com/kelriclink/videoroll) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
