## py-tools

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install in editable mode (required for py_tools.* imports to work)
python -m pip install -e .

# Install with optional extras
python -m pip install -e ".[dev]"       # pytest, flake8, coverage tools
python -m pip install -e ".[datasets]"  # pandas-datareader, python-dotenv
python -m pip install -e ".[ml]"        # scikit-learn, patsy

# Run all tests
python -m pytest

# Run a single test file
python -m pytest tests/test_foo.py

# Run a script-style example
python examples/test_state_space.py

# Quick syntax validation
python -m compileall .
```

The `PY_TOOLS_DATA_DIR` environment variable (or `.env` file) must be set to use dataset loaders.

## Architecture

`py_tools` is a research-oriented Python toolkit for numerical economics, time series, estimation, and dataset loading. The repository root is the `py_tools` package itself (via `pyproject.toml` mapping `py_tools = "."`).

### Package mapping (`pyproject.toml`)

The repo root is `py_tools`, and each subdirectory is a subpackage:

| Import path | Directory |
|---|---|
| `py_tools` | `.` (repo root) |
| `py_tools.time_series` | `time_series/` |
| `py_tools.econometrics` | `econometrics/` |
| `py_tools.bayesian` | `bayesian/` |
| `py_tools.numerical` | `numerical/` |
| `py_tools.stats` | `stats/` |
| `py_tools.econ` | `econ/` |
| `py_tools.data` | `data/` |
| `py_tools.in_out` | `in_out/` |
| `py_tools.config` | `config/` |
| `py_tools.datasets` | `datasets/` |
| `py_tools.compute` | `compute/` |
| `py_tools.plot` | `plot/` |
| `py_tools.utilities` | `utilities/` |
| `py_tools.scraping` | `scraping/` |
| `py_tools.text` | `text/` |

Top-level `__init__.py` uses `__getattr__` for lazy submodule imports. Root-level `.py` files (e.g., `state_space.py`, `mcmc.py`, `time_series.py`) are legacy flat modules that may coexist with their subpackage counterparts during the ongoing refactor.

### Key subpackages

- **`time_series/`** — Kalman filter, state-space models, VAR/BVAR, hidden Markov models. Core TS utility functions in `core.py`.
- **`econometrics/`** — NLS/GMM estimation, bootstrap inference, local projections, high-dimensional fixed effects (`reghdfe.py`).
- **`bayesian/`** — MCMC sampling (`mcmc.py`) and prior distributions.
- **`numerical/`** — Root-finding (Newton), Chebyshev polynomial approximation, general numerical routines.
- **`datasets/`** — Dataset loaders for ~38 economic data sources. Each module exposes `load(**kwargs) -> pd.DataFrame` and declares `DATASET_NAME` and `DESCRIPTION` module-level constants. The `datasets/__init__.py` auto-discovers loaders via AST parsing and builds a registry at import time. Use `datasets.list_datasets()` and `datasets.load_dataset(name, ...)` as the unified entry points.
- **`config/`** — `ConfigRegistry` class (in `registry.py`) for managing named model specification dictionaries with inheritance and deep-merge. `modspec.py` provides model specification helpers.
- **`econ/`** — Discrete choice models, financial math (yield curves, coupon calculations), AIM solver.
- **`data/`** — Data manipulation (`core.py`), collapsing/aggregation (`collapser.py`), dataset matching (`match.py`).

### Ongoing refactor

The repository is mid-way through the plan in `REFACTORING_PLAN.md`. Many root-level `.py` files (e.g., `kalman.py`, `state_space.py`, `vector_autoregression.py`, `mcmc.py`) are legacy flat modules that have been or are being migrated into subpackages. Both the old flat path and the new subpackage path may exist simultaneously during migration.

### Dataset loader convention

Each dataset module in `datasets/` should declare:
```python
DATASET_NAME = "descriptive_name"   # used as registry key
DESCRIPTION = "One-line description."
def load(data_dir=None, **kwargs) -> pd.DataFrame: ...
```
The `datasets/__init__.py` picks these up automatically via AST parsing — no manual registration needed.

### Testing

Tests live in `tests/` and use pytest. The `examples/` directory holds script-style demos that are excluded from pytest discovery (`norecursedirs` in `pyproject.toml`). For numerical assertions use `np.allclose(..., atol=1e-8)` with explicit tolerances.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dgreenwald)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dgreenwald)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
