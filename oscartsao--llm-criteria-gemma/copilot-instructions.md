## llm-criteria-gemma

> Use this guide to keep the Gemma NLI encoder consistent.

# Repository Guidelines

Use this guide to keep the Gemma NLI encoder consistent.

## Project Structure & Module Organization
- `src/models/` holds the core encoder (`gemma_encoder.py`), poolers, and DoRA layers used during training.
- `src/data/` contains dataset loaders, DSM-5 utilities, and CV helpers; raw assets live in `data/` and should keep derived files beside their generating scripts.
- `src/training/` provides entry scripts for binary NLI training and evaluation.
- `conf/` stores Hydra defaults and overrides; never hard-code experiment tweaks in code when they belong in `conf/`.
- `outputs/` collects checkpoints and metrics, so avoid editing its contents by hand.
- Tests live under `tests/` and documentation under `docs/`.

## Build, Test & Development Commands
- `pip install -r requirements.txt` installs runtime + dev dependencies.
- `make train-nli` or `python src/training/train_nli_binary.py` launches training with the default Gemma-3 1B checkpoint.
- `make evaluate-nli` or `python src/training/evaluate_nli_binary.py --checkpoint <path>` scores a saved model.
- `make test` (preferred) or `pytest -m "not slow"` runs the automated suite; `make check-data` verifies dataset presence before experiments.

## Coding Style & Naming Conventions
- Python 3.8+ with 4-space indentation, descriptive snake_case for functions/modules, PascalCase for classes, and UPPER_SNAKE_CASE for constants (e.g., `DSM5_LABELS`).
- Run `black .` (line length 100) and `isort .` before pushing; configs live in `pyproject.toml`.
- Prefer type hints on public functions and keep docstrings action-oriented.
- Run `mypy src` when touching model interfaces.

## Testing Guidelines
- All tests use `pytest` with the defaults from `pyproject.toml`; new files must follow `test_*.py` naming and live inside `tests/`.
- Target `--cov=src` ≥ 80%; add focused unit tests for new poolers, dataset transforms, or training utilities.
- Mark GPU-heavy or long experiments with `@pytest.mark.slow` or `@pytest.mark.gpu` and ensure the default suite skips them.

## Commit & Pull Request Guidelines
- Follow the repo’s short imperative commit style (e.g., `Fix deprecation warnings in training script`).
- Each PR should describe motivation, approach, and testing evidence; link related issues and attach sample metrics or tensorboard screenshots when relevant.
- Confirm `make test` and linting complete before requesting review, and mention any skipped commands with justification.

## Configuration & Data Notes
- Keep sensitive tokens (HuggingFace, MLflow) in environment variables rather than config files; reference them via Hydra overrides instead of committing plaintext keys.
- Use `conf/experiment/<name>.yaml` for overrides, document new knobs in `README.md` or `docs/`, and keep checkpoints/datasets in `outputs/` or external storage instead of git.

---
> Source: [OscarTsao/LLM_Criteria_Gemma](https://github.com/OscarTsao/LLM_Criteria_Gemma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
