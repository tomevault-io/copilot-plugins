## e-project-big-data

> - Backend Flask services live in `backend/`; blueprints go under `backend/api/`, configuration in `backend/config/`, and HTTP entrypoint in `backend/app.py`.

# Repository Guidelines

## Project Structure & Module Organization
- Backend Flask services live in `backend/`; blueprints go under `backend/api/`, configuration in `backend/config/`, and HTTP entrypoint in `backend/app.py`.
- Frontend assets (static HTML, JS, CSS) live in `frontend/` and can be opened directly via `frontend/index.html`.
- Data engineering utilities reside in `hadoop_scripts/` (top-level scripts plus `jobs/` for MapReduce code).
- Machine learning experiments live in `ml_models/src/`; notebooks or exploratory work should stay in `ml_models/notebooks/`.
- Shared assets such as sample data (`sensor_data.json`, `weather_station_data.csv`) stay at the repo root so every component can reference them.

## Build, Test, and Development Commands
- `uv sync && source .venv/bin/activate` – creates or updates the project virtual environment using `pyproject.toml`.
- `uv run python backend/app.py` – launches the Flask API locally at `http://127.0.0.1:5000`.
- `uv run python hadoop_scripts/ingest_data.py` – produces `dummy_climate_data.csv` for downstream jobs.
- `uv run python hadoop_scripts/jobs/climate_analysis.py dummy_climate_data.csv` – runs the MapReduce job with `mrjob`.
- `uv run python ml_models/src/climate_model.py` – retrains the regression model and prints anomaly output.

## Coding Style & Naming Conventions
- Python: follow PEP 8 with 4-space indents, snake_case for functions/variables, PascalCase for classes, and include type hints where practical.
- JavaScript/CSS: keep files in `frontend/src/`, use camelCase for functions, kebab-case for CSS classes, and prefer ES6 features.
- Config files should be declarative (e.g., `pyproject.toml`, `.env.example`) and never embed secrets; rely on `Config` in `backend/config` to read environment variables.

## Testing Guidelines
- Add automated tests under `backend/tests/` using the standard library’s `unittest` (import `unittest` and run `uv run python -m unittest discover backend/tests`).
- Mirror API blueprint names in your test module filenames (e.g., `test_user_api.py`) and keep fixtures small and deterministic.
- When touching ML or Hadoop code, add smoke tests or sample-run scripts that verify CLI entry points succeed against generated dummy data.

## Commit & Pull Request Guidelines
- Use present-tense, imperative commit subjects (e.g., “Add user login blueprint validation”) and include context in the body when multiple modules change.
- Reference related GitHub issues with `Fixes #123` or `Refs #456` inside commits or PR descriptions.
- PRs should summarize the change, list test evidence (`uv run python backend/app.py` / `uv run python -m unittest …`), and include screenshots or sample responses when front-end or API behavior changes.

---
> Source: [eDeveloper132/e-project_big-data](https://github.com/eDeveloper132/e-project_big-data) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
