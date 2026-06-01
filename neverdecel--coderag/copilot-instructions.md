## coderag

> - `coderag/`: Core library (`config.py`, `embeddings.py`, `index.py`, `search.py`, `monitor.py`).

# Repository Guidelines

## Project Structure & Module Organization
- `coderag/`: Core library (`config.py`, `embeddings.py`, `index.py`, `search.py`, `monitor.py`).
- `app.py`: Streamlit UI. `main.py`: backend/indexer. `prompt_flow.py`: RAG orchestration.
- `scripts/`: Utilities (e.g., `initialize_index.py`, `run_monitor.py`).
- `tests/`: Minimal checks (e.g., `test_faiss.py`).
- `example.env` → copy to `.env` for local secrets; CI lives in `.github/`.

## Build, Test, and Development Commands
- Create env: `python -m venv venv && source venv/bin/activate`.
- Install deps: `pip install -r requirements.txt`.
- Run backend: `python main.py` (indexes and watches `WATCHED_DIR`).
- Run UI: `streamlit run app.py`.
- Quick test: `python tests/test_faiss.py` (FAISS round‑trip sanity check).
- Quality suite: `pre-commit run --all-files` (black, isort, flake8, mypy, basics).

## Coding Style & Naming Conventions
- Formatting: Black (88 cols), isort profile "black"; run `black . && isort .`.
- Linting: flake8 with `--ignore=E203,W503` to match Black.
- Typing: mypy (py311 target; ignore missing imports OK). Prefer typed signatures and docstrings.
- Indentation: 4 spaces. Names: `snake_case` for files/functions, `PascalCase` for classes, constants `UPPER_SNAKE`.
- Imports: first‑party module is `coderag` (see `pyproject.toml`).

## Testing Guidelines
- Place tests in `tests/` as `test_*.py`. Keep unit tests deterministic; mock OpenAI calls where possible.
- Run directly (`python tests/test_faiss.py`) or with pytest if available (`pytest -q`).
- Ensure `.env` or env vars provide `OPENAI_API_KEY` for integration tests; avoid hitting rate limits in CI.

## Commit & Pull Request Guidelines
- Use Conventional Commits seen in history: `feat:`, `fix:`, `docs:`, `ci:`, `refactor:`, `simplify:`.
- Before pushing: `pre-commit run --all-files` and update docs when behavior changes.
- PRs: clear description, linked issues, steps to validate; include screenshots/GIFs for UI changes; note config changes (`.env`).

## Security & Configuration Tips
- Never commit secrets. Start with `cp example.env .env`; set `OPENAI_API_KEY`, `WATCHED_DIR`, `FAISS_INDEX_FILE`.
- Avoid logging sensitive data. Regenerate the FAISS index if dimensions or models change (`python scripts/initialize_index.py`).

---
> Source: [Neverdecel/CodeRAG](https://github.com/Neverdecel/CodeRAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
