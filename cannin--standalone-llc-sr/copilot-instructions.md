## standalone-llc-sr

> - `gene_regulatory_dynamics.py` is the end-to-end LLC pipeline (topology build → simulation → NN training → symbolic regression). Treat it as the single source of truth for hyperparameters and data flow.

# Repository Guidelines

## Project Structure & Module Organization
- `gene_regulatory_dynamics.py` is the end-to-end LLC pipeline (topology build → simulation → NN training → symbolic regression). Treat it as the single source of truth for hyperparameters and data flow.
- `original_llc_config.toml` holds Dynaconf environments (default `simple`). Switch configs via `ENV_FOR_DYNACONF=<env> python gene_regulatory_dynamics.py`.
- `plot_nn12.py` rebuilds trajectories from symbolic NN1/NN2 expressions under `results_*/symbolic_regression/` and writes comparison plots to `results_*/plots/`.
- `nesymres/` vendors the symbolic-regression dependencies; `weights/` stores pretrained checkpoints and metadata consumed by the pipeline; every run writes artifacts under `results/`.

## Build, Test, and Development Commands
- `uv run python gene_regulatory_dynamics.py` (or plain `python`) runs the full workflow and refreshes every artifact in `results/`.
- `ENV_FOR_DYNACONF=test1 python gene_regulatory_dynamics.py` selects a config environment; copy the `[test1]` block in `dynamics_config.toml` to create new presets.
- `python plot_nn12.py` recreates trajectories using the latest symbolic expressions and emits RMSE per node plus `results_*/plots/gene_regulatory_nn12_vs_truth.png`.

## Coding Style & Naming Conventions
- Follow PEP 8 with 4-space indentation and keep modules ASCII-only unless upstream data requires otherwise.
- Prefer type hints, explicit Google-style docstrings (Args/Returns), and concise comments only when logic is non-obvious.
- Comment code sections for educational purposes especially function purpose.
- Configuration constants should live near the top of `gene_regulatory_dynamics.py`; favor descriptive snake_case for variables and lowercase filenames (e.g., `power_law_result_dict.json`).

## Testing Guidelines
- There is no standalone unit-test suite; validation relies on rerunning `gene_regulatory_dynamics.py` and checking generated plots/CSVs.

## Commit & Pull Request Guidelines
- Follow the existing history: short, capitalized, imperative summaries (e.g., `Update gene regulatory dynamics script to use original configuration file`).
- Each PR should describe the change set, note any configuration/env impacts, and include before/after evidence (paths to `results/...` artifacts or log excerpts). Link associated issues and mention reviewers for modeling changes.
- When modifying generated outputs, highlight whether they stem from code or config adjustments to keep reviewers focused on intentional diffs.

---
> Source: [cannin/standalone_llc_sr](https://github.com/cannin/standalone_llc_sr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
