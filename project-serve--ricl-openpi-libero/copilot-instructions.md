## ricl-openpi-libero

> Core code lives in `src/openpi/` with domain folders such as `models/`, `policies/`, `training/`, `shared/`, and `serving/`. Reusable client/runtime code is in `packages/openpi-client/src/openpi_client/`. Operational entry points are under `scripts/` (for example `train_pi0_fast_ricl.py`, `serve_policy_ricl.py`). Data preparation utilities are in `preprocessing/`. Integration examples for robots and environments are in `examples/`. Small tracked assets are in `assets/`; large checkpoints and collected demos are intentionally gitignored.

# Repository Guidelines

## Project Structure & Module Organization
Core code lives in `src/openpi/` with domain folders such as `models/`, `policies/`, `training/`, `shared/`, and `serving/`. Reusable client/runtime code is in `packages/openpi-client/src/openpi_client/`. Operational entry points are under `scripts/` (for example `train_pi0_fast_ricl.py`, `serve_policy_ricl.py`). Data preparation utilities are in `preprocessing/`. Integration examples for robots and environments are in `examples/`. Small tracked assets are in `assets/`; large checkpoints and collected demos are intentionally gitignored.

## Build, Test, and Development Commands
- `GIT_LFS_SKIP_SMUDGE=1 uv sync`: install workspace dependencies without downloading large LFS payloads.
- `source .venv/bin/activate`: activate the local virtual environment.
- `uv run pytest`: run tests across `src`, `scripts`, and `packages`.
- `uv run pytest -m "not manual"`: run CI-friendly tests only.
- `uv run ruff check .`: lint code (configured in `pyproject.toml`).
- `uv run ruff format .`: apply formatting.
- `pre-commit run --all-files`: run configured hooks (`uv-lock`, Ruff) before pushing.

## Coding Style & Naming Conventions
Use Python with 4-space indentation and a maximum line length of 120. Follow Ruff defaults plus repo rules in `pyproject.toml`. Prefer `snake_case` for files, functions, and variables; `PascalCase` for classes; `UPPER_SNAKE_CASE` for constants. Keep module boundaries clear (`training` logic in `src/openpi/training/`, policy logic in `src/openpi/policies/`).

## Testing Guidelines
Place tests as `*_test.py`, typically beside the code they validate. Use `pytest` fixtures/parametrization for deterministic unit coverage. Mark long-running or environment-dependent tests with `@pytest.mark.manual`. During development, run focused tests (example: `uv run pytest src/openpi/models/model_test.py`), then run the full suite before opening a PR.

## Commit & Pull Request Guidelines
Recent history uses short, single-line summaries (example: `ricl_openpi_edited for capstone`). Keep commit subjects concise and descriptive; add scope when helpful (example: `training: fix demo-group retrieval path`). PRs should include purpose, key changes, validation commands run, and any data/checkpoint assumptions. Link related issues/discussions and include logs or screenshots when behavior changes are observable.

## Data & Configuration Tips
Do not commit generated artifacts in ignored paths such as `checkpoints/`, `wandb/`, `preprocessing/collected_demos_training/`, or `preprocessing/collected_demos/`. Keep collected-task folder naming consistent with preprocessing expectations: `{YYYY-MM-DD}_{task_prompt}`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/project-SeRVe)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/project-SeRVe)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
