## ai-rules-sync

> You are working in a Python 3.12 FastAPI service.

You are working in a Python 3.12 FastAPI service.

- Install deps with `uv sync`.
- Run the dev server with `uv run fastapi dev`.
- Run tests with `uv run pytest`.
- Format and lint with `uv run ruff check --fix`.

Conventions:
- Use type hints everywhere; the project runs mypy in strict mode.
- Pydantic v2 models for all request/response bodies.
- Keep route handlers thin; put business logic in `app/services/`.

Never commit secrets. Do not edit anything under `app/migrations/`.

---
> Source: [PanisHandsome/ai-rules-sync](https://github.com/PanisHandsome/ai-rules-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
