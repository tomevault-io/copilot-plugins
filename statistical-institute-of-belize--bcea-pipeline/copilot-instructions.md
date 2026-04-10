## bcea-pipeline

> Core pipeline code lives in `src/`, split into `preprocess.py`, `model.py`, `predict.py`, and `evaluate.py` orchestrated by `main.py`. Shared helpers sit in `src/utils.py`. Data assets belong in `data/` (`raw/`, `processed/`, `mappings/`, `review/`), trained checkpoints in `models/`, and runtime traces in `logs/`. The FastAPI layer is under `api/` (routers in `api/routers/`), while API docs live in `docs/`. Tune behaviour through `config.yaml`.

# Repository Guidelines

## Project Structure & Module Organization
Core pipeline code lives in `src/`, split into `preprocess.py`, `model.py`, `predict.py`, and `evaluate.py` orchestrated by `main.py`. Shared helpers sit in `src/utils.py`. Data assets belong in `data/` (`raw/`, `processed/`, `mappings/`, `review/`), trained checkpoints in `models/`, and runtime traces in `logs/`. The FastAPI layer is under `api/` (routers in `api/routers/`), while API docs live in `docs/`. Tune behaviour through `config.yaml`.

## Build, Test, and Development Commands
- `python -m venv venv && source venv/bin/activate` — create or refresh the local virtualenv.
- `pip install -r requirements.txt` — install pipeline and API dependencies.
- `python main.py --config config.yaml` — run preprocessing, training, and evaluation end to end.
- `python main.py --skip-training --evaluate` — reuse the latest model for metrics only.
- `python main.py --skip-training --input data/new_data.csv` — score new records without retraining.
- `uvicorn api.main:app --reload --port 8000` — launch the REST API (mirrors `python api_server.py`).

## Coding Style & Naming Conventions
Follow PEP 8 with four-space indentation and descriptive snake_case identifiers; reserve PascalCase for classes like `BCEADataset`. Mirror the docstring format used in `src/model.py` for new public functions and keep logging consistent via `rich` utilities in `src/utils.py`. Configuration values belong in YAML rather than hard-coded constants.

## Testing Guidelines
Use `python src/evaluate.py` (or the `--evaluate` CLI flag) to validate metrics against `data/processed/test.csv` before shipping changes. When adding automated checks, prefer `pytest`, store files under a new `tests/` directory, and name them `test_<module>.py`. Provide lightweight fixtures or sample CSV rows so preprocessing and label mapping branches are exercised.

## Commit & Pull Request Guidelines
Git history favours short, action-oriented messages (`API layer`, `improved eval`); keep titles under 60 characters and use the imperative mood. Squash noisy WIP commits locally. Pull requests should summarize pipeline or API impact, list datasets touched, note evaluation results or API responses, and link related issues. Include screenshots of new API responses when updating request/response schemas.

## Security & Configuration Tips
Do not commit real datasets or secrets in `config.yaml`; store sensitive paths via environment variables and reference them in the YAML with placeholders. Ensure `data/raw/` uploads are anonymized, and review `models/` artifacts for export compliance before tagging a release.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Statistical-Institute-Of-Belize)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Statistical-Institute-Of-Belize)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
