## cwe-chatbot

> - `apps/cwe_ingestion/`: CWE downloader, parser, embedders (Gemini/local), and Postgres stores.

# Repository Guidelines

## Project Structure & Module Organization
- `apps/cwe_ingestion/`: CWE downloader, parser, embedders (Gemini/local), and Postgres stores.
- `apps/chatbot/src/`: Chainlit app (RAG, retrieval, formatting, security utilities).
- `shared/`: Reusable helpers.
- `tests/`, `apps/**/tests/`: Unit/integration tests.
- `pyproject.toml`: Poetry config, dependencies, tools (ruff, black, mypy).

## Build, Test, and Development Commands
- Install: `poetry install`
- Lint (ruff): `poetry run ruff check .`
- Format (black): `poetry run black .`
- Type check (mypy): `poetry run mypy .`
- Tests (default targets): `poetry run pytest -q`
- Ingestion CLI (examples):
  - `poetry run python apps/cwe_ingestion/cli.py ingest -e gemini`
  - `poetry run python apps/cwe_ingestion/cli.py query -q "sql injection" --hybrid`

## Coding Style & Naming Conventions
- Python 3.10+. Use PEP 8; 4‑space indentation.
- Naming: `snake_case` for files/functions, `CamelCase` for classes, constants `UPPER_SNAKE`.
- Keep functions focused; prefer explicit types for public APIs.
- Tools: ruff (lint/imports), black (format, 88 cols), mypy (type hints).

## Testing Guidelines
- Framework: pytest (+ coverage). Tests live in `tests/` and under app folders.
- Name tests `test_*.py`; functions `test_*`.
- Run all: `poetry run pytest`.
- Targeted runs: `poetry run pytest apps/cwe_ingestion/tests -q`.

## Commit & Pull Request Guidelines
- Commits: imperative mood, concise scope (e.g., `fix: rrf hybrid query in chunk store`).
- Include why + what; reference issues (`Closes #123`) when applicable.
- PRs: clear description, screenshots/logs for UX/CLI changes, note migrations.
- Keep diffs focused; include tests for bug fixes and features.

## Security & Configuration Tips
- Never commit secrets. Use `.env.example` as a guide.
- Common env vars: `DATABASE_URL`, `LOCAL_DATABASE_URL`, `PROD_DATABASE_URL`, `GEMINI_API_KEY`.
- Gemini embeddings are 3072‑D by default; ensure pgvector columns match.
- For tokenizer‑aware chunking, install `tiktoken` (already listed in Poetry).

## Quick Architecture Notes
- Ingestion: parse → chunk → embed (Gemini/local) → store (pgvector) → hybrid query (RRF over vector/FTS/alias).
- Chatbot: Hybrid RAG over Postgres + formatting/security layers in `apps/chatbot/src`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CWE-ChatBot)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CWE-ChatBot)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
