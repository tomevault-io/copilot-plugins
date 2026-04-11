## elmy

> Python utilities live in `src/utils/` and cover preprocessing (`process_data.py`), feature engineering (`lag_data.py`), evaluation (`weighted_accuracy.py`, `weighted_accuracy_and_tools.py`), and visualization (`plot_learning_curves.py`). Align new helpers with this layout and keep configuration constants near their consumers. Mirrors of these modules should sit under `tests/` (e.g., `tests/test_weighted_accuracy.py`) to preserve one-to-one coverage. Exploratory notebooks belong in `notebooks/` and follow `YYYYMMDD_topic.ipynb`; promote any reusable code into `src/utils/` before shipping. Large datasets stay in a gitignored `data/` subtree outside version control.

# Repository Guidelines

## Project Structure & Module Organization
Python utilities live in `src/utils/` and cover preprocessing (`process_data.py`), feature engineering (`lag_data.py`), evaluation (`weighted_accuracy.py`, `weighted_accuracy_and_tools.py`), and visualization (`plot_learning_curves.py`). Align new helpers with this layout and keep configuration constants near their consumers. Mirrors of these modules should sit under `tests/` (e.g., `tests/test_weighted_accuracy.py`) to preserve one-to-one coverage. Exploratory notebooks belong in `notebooks/` and follow `YYYYMMDD_topic.ipynb`; promote any reusable code into `src/utils/` before shipping. Large datasets stay in a gitignored `data/` subtree outside version control.

## Build, Test, and Development Commands
Create an isolated environment before editing:
```bash
python -m venv env
source env/bin/activate
pip install -r requirements.txt
```
Run preprocessing via `python src/utils/process_data.py --input data/raw/source.csv --output data/interim/clean.csv`. Launch visual diagnostics with `python src/utils/plot_learning_curves.py --run-id 2024-01-15`. Execute the suite from the repo root using `pytest`; add `-k pattern` to target individual modules.

## Coding Style & Naming Conventions
Follow PEP 8 with 4-space indents and snake_case for functions, variables, and modules; reserve CamelCase for classes. Prefer typed dataclasses or `NamedTuple`s for structured returns. Use f-strings for formatting, document public functions with concise Google-style docstrings, and keep top-level constants in UPPER_SNAKE_CASE. Apply domain-friendly names that mirror data schema fields.

## Testing Guidelines
Tests rely on `pytest` with parametrized cases to cover multiple forecast horizons and weighting behaviors. Keep fixtures lightweight and deterministic, refreshing them when the input schema shifts. Target ≥80% coverage, and fail fast on masking warnings by running `pytest -W error`. Name files `test_<module>.py` to mirror `src/utils` structure.

## Commit & Pull Request Guidelines
Commits use short, imperative subjects like `update lag data logic`, limited to 50 characters. Reference related notebook runs or dataset snapshots in the body when relevant. Pull requests should state motivation, list runnable commands (env setup, scripts, tests), note dataset versions, and include before/after metrics or plots when model behavior changes. Link challenge tickets so future reviewers can trace decisions.

## Data & Notebook Tips
Cache raw Challenge Data downloads in `data/raw/` (gitignored) and write derived artifacts to `data/interim/` with timestamps for reproducibility. Signal notebook readiness by summarizing conclusions at the top and logging reproduction commands; keep them narrative and move production code into modules.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JonasBlx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JonasBlx)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
