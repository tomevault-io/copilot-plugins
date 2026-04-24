## ban-501-spring-2026-icwip

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Course materials for BAN-501 featuring interactive notebooks built with [marimo](https://marimo.io/). The notebooks cover machine learning topics in sequence:

1. `1-linear-regression.py` — Hill-climbing optimization on synthetic data, statsmodels verification
2. `2-lasso-ridge-regression.py` — Ridge, Lasso, Elastic Net on Ames Housing (regression)
3. `3-logistic-regression.py` — Binary classification with statsmodels on bank marketing data
4. `4-tree-based-models.py` — Decision trees and random forests with GridSearchCV and Optuna, logistic regression comparison, permutation importance
5. `5-xgboost.py` — XGBoost classification with Optuna hyperparameter tuning, comparison against Random Forest
6. `6-concurrency.py` — Python concurrency concepts (threads vs processes), imports helpers from `_concurrency_helpers.py`
7. `7-dimensionality-reduction-demo.py` — PCA and PaCMAP demo on MNIST, Random Forest on reduced features
8. `8-dimensionality-reduction.py` — Full dimensionality reduction pipeline on bank marketing data with PCA, PaCMAP, and classification
9. `9-clustering.py` — K-Means and HDBSCAN on synthetic data (blobs, circles), density-based vs centroid-based comparison
10. `10-pytorch-introduction.py` — Feedforward neural networks on MNIST with PyTorch
11. `11-pytorch-cnn.py` — CNNs vs Feedforward Networks on MNIST
12. `12-transfer-learning.py` — Fine-tuning pre-trained ResNet50 on selfie/non-selfie classification, model persistence

Supporting files:
- `_concurrency_helpers.py` — Shared functions (`is_prime`, `simulate_io_task`) and constants for notebook 6
- `custom_RF_complete.py` — Marimo notebook implementing Random Forest from scratch with statsmodels, tqdm, and manual bagging

Concept slides live in `concept-slides/beamer/` as LaTeX Beamer source, with compiled PDFs in `concept-slides/`.

## Environment Setup

This project uses [pixi](https://pixi.sh/) for environment management with Python 3.14.

```bash
pixi install
pixi run python <script.py>
```

To add packages: `pixi add PACKAGE_NAME` (do not manually edit `pixi.toml` dependencies).

## Running Notebooks

Marimo notebooks are Python files with `@app.cell` decorators.

**Non-interactive execution** (for scripts/CI):
```bash
MPLBACKEND=Agg pixi run python 1-linear-regression.py
```

**Interactive editing**:
```bash
pixi run marimo edit 1-linear-regression.py
```

## Notebook Architecture

Each notebook follows a consistent pattern:

1. **Single import cell** — All imports in one `@app.cell def _():` block, returning every name used by other cells. Includes `sns.set_style("whitegrid")` and `optuna.logging.set_verbosity(optuna.logging.WARNING)` where applicable.
2. **Markdown cells** — `@app.cell(hide_code=True)` with `mo.md(r"""...""")` for section explanations with LaTeX math.
3. **Code cells** — Receive dependencies via function signature (e.g., `def _(pl, plt, X_train):`). Return any variables needed downstream.
4. **Empty trailing cell** — Each notebook ends with an empty `@app.cell` before `app.run()`.

Notebooks 2-5 and 8 share a common pipeline pattern on the bank marketing dataset: load parquet → select features → train/test split with stratification → one-hot encode categoricals → fit model → evaluate with AUC/confusion matrix. Notebooks 7-8 use MNIST and bank marketing data respectively with StandardScaler preprocessing and PCA/PaCMAP for dimensionality reduction.

## Marimo Notebook Conventions

Marimo builds a reactive dependency graph from variable names. Follow these patterns:

**Underscore prefix for temporary variables:**
- Loop variables: `for _i, _item in enumerate(...)`
- Intermediate values: `_temp = load(); result = process(_temp)`
- Figure/axes: `_fig, _ax = plt.subplots()`
- File handles: `with open(path) as _f:`

**Import pattern:**
```python
# First cell - define imports once
@app.cell
def _():
    import polars as pl
    import matplotlib.pyplot as plt
    return pl, plt

# Other cells receive via function signature
@app.cell
def _(pl, plt):
    _df = pl.read_csv("data.csv")
    ...
```

**Avoid:**
- Re-importing modules in multiple cells
- Double-assigning the same variable name (use method chaining or `_` intermediates)
- Code outside `@app.cell` decorators

## Code Style

- Data manipulation: polars (not pandas), except when passing to statsmodels (which requires pandas)
- Visualization: matplotlib + seaborn with `sns.set_style("whitegrid")`, `edgecolor='k'` on barplots
- Use keyword arguments for function calls with multiple parameters
- Set random seeds explicitly for reproducibility (`random_state=42`, `seed=42`)

## Data

```
data/
├── regression/train.parquet                              # Ames Housing dataset
├── classification/playground-series-s5e8/train.parquet   # Bank marketing dataset
├── MNIST/
│   ├── mnist_features.parquet                            # 784 pixel features
│   └── mnist_target.parquet                              # Digit labels
└── selfie_data/
    ├── Selfie/                                           # 3,931 selfie JPGs
    └── NonSelfie/                                        # 3,931 non-selfie JPGs
```

## Concept Slides

LaTeX Beamer presentations in `concept-slides/beamer/`. Compile with:
```bash
cd concept-slides/beamer && pdflatex <filename>.tex
```
Generated auxiliary files (`.aux`, `.log`, `.out`, etc.) are gitignored.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nkfreeman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
