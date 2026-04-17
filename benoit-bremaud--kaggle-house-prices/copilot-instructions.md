## kaggle-house-prices

> Kaggle competition project — data science / machine learning.

# Claude Code — Kaggle Project Configuration

## Project Type

Kaggle competition project — data science / machine learning.

## Stack

- **Python** 3.12+ with **uv** as package manager
- **Jupyter** notebooks for analysis and experimentation
- **ruff** for linting and formatting
- **pre-commit** + **nbstripout** for Git hooks
- **scikit-learn**, **pandas**, **numpy**, **matplotlib**, **seaborn** for data science
- **xgboost**, **lightgbm** for gradient boosting models

## Workflow

- `make setup` — Initialize the environment (venv, dependencies, hooks)
- `make notebook` — Start Jupyter Lab
- `make lint` — Check code quality
- `make format` — Auto-format code
- `make data COMPETITION=name` — Download competition data
- `make submit COMPETITION=name FILE=path` — Submit predictions

## Git Conventions

- **Branch naming:** `feat/{experiment-name}` (e.g., `feat/random-forest-baseline`)
- **Commit format:** Conventional Commits — `<type>(<scope>): <description>`
- **Branch protection:** PRs required to merge into `main`, no force push
- **CI checks required:** `Lint & Format` (ruff) and `Tests` (pytest) must pass before merge
- **Notebook outputs:** Automatically stripped by nbstripout on commit

## Project Structure

- `notebooks/` — Jupyter notebooks (main analysis)
- `src/` — Reusable Python modules (utility functions, feature engineering)
- `data/raw/` — Original competition data (gitignored, download via `make data`)
- `data/processed/` — Cleaned/transformed data (gitignored)
- `outputs/models/` — Saved trained models (gitignored)
- `outputs/submissions/` — Submission CSVs and score log (tracked in Git)

## Important Rules

- **Never commit notebook outputs** — nbstripout handles this via pre-commit hook
- **Never commit raw data** — data/ is gitignored; download via Kaggle API or manually
- **Always use `src/`** for reusable code extracted from notebooks
- **Track submission scores** in `outputs/submissions/log.md`
- **No `any` type** — not applicable here (Python project) but maintain type hints where practical
- **No hardcoded paths** — use the `Path` constants defined in notebook section 2 (Configuration)

## Decisions

All architectural decisions are documented in:
- `DECISIONS.md` (project-specific decisions)
- `../DECISIONS.md` (global decisions for all Kaggle projects)

Read these files for full context on stack choices, conventions, and rationale.

## Labels for Issues

When creating issues, use the DS-oriented label system:
- **Type:** `type:eda`, `type:data-cleaning`, `type:feature-eng`, `type:modeling`, `type:evaluation`, `type:submission`, `type:bug`, `type:refactor`, `type:docs`, `type:setup`, `type:experiment`
- **Area:** `area:notebook`, `area:src`, `area:data`, `area:model`, `area:infra`
- **Priority:** `priority:critical`, `priority:high`, `priority:medium`, `priority:low`
- **Status:** `status:blocked`, `status:needs-review`, `status:wontfix`
- **Meta:** `good-first-issue`

Assign multiple relevant labels per issue for effective filtering.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/benoit-bremaud) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
