## anyrouter-transparent-proxy

> - `backend/` FastAPI proxy service; entry `app.py`, settings `config.py`, logic in `services/`, routes in `routers/`, helpers in `utils/`.

# Repository Guidelines

## Project Structure & Module Organization
- `backend/` FastAPI proxy service; entry `app.py`, settings `config.py`, logic in `services/`, routes in `routers/`, helpers in `utils/`.
- `frontend/` Vue 3 + TypeScript dashboard; pages in `views/`, shared UI in `components/`, API wrappers in `services/`, stores in `stores/`, composables in `composables/`, styles in `style.css`.
- `static/` holds built frontend assets; `env/` stores deploy configs like `.env.headers.json`; `docs/`, `scripts/`, `tests/` contain docs, tooling, and Python checks.

## Build, Test, and Development Commands
- Backend setup and run:  
  ```bash
  pip install -r backend/requirements.txt
  python -m backend.app              # start server at :8088
  # or hot reload
  uvicorn backend.app:app --reload --port 8088
  ```
- Frontend workflow (prefers pnpm because `pnpm-lock.yaml` is present):  
  ```bash
  cd frontend
  pnpm install
  pnpm dev                           # Vite dev server
  pnpm build                         # outputs to ../static for backend to serve
  pnpm preview                       # preview production build
  pnpm type-check                    # TypeScript only
  ```
- Tests and checks (run from repo root):  
  ```bash
  python tests/test_encoding.py      # encoding utility coverage
  python tests/test_imports.py       # module import/syntax sanity
  python tests/test_integration.py   # end-to-end structure chain
  ```
- Docker: `docker-compose up -d` to start full stack; rebuild with `docker-compose up -d --build --remove-orphans`.

## Coding Style & Naming Conventions
- Python: 4-space indent, type hints, routing in `routers/`, request shaping in `services/`; avoid global mutable state; load config from `.env`. Helpers belong in `utils/`.
- Vue/TS: 2-space indent, `<script setup lang="ts">`, PascalCase component files (`Dashboard.vue`), composables `useX`, Pinia stores `useNameStore`, API clients under `services/` returning typed results. Prefer single quotes and no semicolons.
- Built assets in `static/` stay untracked; keep env values in `env/` and `.env.example`.

## Testing Guidelines
- Keep backend checks lightweight under `tests/` with print-based reporting.
- Extend `test_imports.py` for import coverage and add focused scripts similar to `test_encoding.py`.
- Frontend changes should at least pass `pnpm type-check`; add targeted tests for logic-heavy modules.

## Commit & Pull Request Guidelines
- Use Conventional Commit prefixes seen in history: `feat: ...`, `feat(scope): ...`, `docs: ...`, `test: ...`; keep messages concise.
- PRs should summarize changes, list manual test steps or scripts run, link issues, and include screenshots/GIFs for UI updates.

## Security & Configuration Tips
- Never commit secrets; keep runtime values in `.env` and header overrides in `env/.env.headers.json` (copy from `.env.headers.json.example`). Document proxy variables (`HTTP_PROXY`, `HTTPS_PROXY`) when needed.
- Validate upstream targets via `API_BASE_URL`; keep admin panel (`/admin`) behind appropriate network controls.

---
> Source: [RebornQ/AnyRouter-Transparent-Proxy](https://github.com/RebornQ/AnyRouter-Transparent-Proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
