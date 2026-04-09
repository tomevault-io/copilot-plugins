## metaboanalyst-clone

> Build a **desktop Python GUI application** (PySide6) that fully replicates MetaboAnalyst 6.0's standardization pipeline and statistical visualization. The goal is to open the "black box" — every algorithm must be transparent and documented.

# CLAUDE.md — MetaboAnalyst 6.0 Python Replication

## Project Overview

Build a **desktop Python GUI application** (PySide6) that fully replicates MetaboAnalyst 6.0's standardization pipeline and statistical visualization. The goal is to open the "black box" — every algorithm must be transparent and documented.

**Source of truth:** `xia-lab/MetaboAnalystR` GitHub repo (`general_proc_utils.R`, `general_norm_utils.R`, `stats_univariates.R`, `stats_chemometrics.R`)

**Reference:** Pang et al. (2024) *Nucleic Acids Research*, 52(W1), W398–W406.

---

## Tech Stack

- **Language:** Python 3.10+
- **GUI:** PySide6 (with matplotlib embedded via `FigureCanvasQTAgg`)
- **Core:** numpy, pandas, scipy, scikit-learn, statsmodels
- **Visualization:** matplotlib, seaborn, plotly (3D PCA only), adjustText
- **Specialized:** qnorm (quantile normalization), pyopls (OPLS-DA), fancyimpute (SVD impute), pyppca (PPCA impute)

```bash
pip install numpy pandas scipy scikit-learn statsmodels matplotlib seaborn plotly adjustText PySide6 fancyimpute pyppca qnorm pyopls
```

---

## Detailed Specs (in `docs/specs/`)

| File | Contents |
|---|---|
| [`01-algorithms.md`](docs/specs/01-algorithms.md) | All core processing algorithms: glog transform, missing values, filtering, normalization, transformation, scaling, R↔Python mapping |
| [`02-visualization.md`](docs/specs/02-visualization.md) | PCA, volcano, heatmap, VIP, boxplot, density plot specifications |
| [`03-gui.md`](docs/specs/03-gui.md) | GUI layout, tab descriptions, theming, widgets, threading, undo/redo |
| [`04-i18n.md`](docs/specs/04-i18n.md) | Internationalization: Qt translation system, CJK fonts, translation table |
| [`05-deployment.md`](docs/specs/05-deployment.md) | PyInstaller, Inno Setup, macOS signing, GitHub Actions CI/CD |
| [`06-compatibility.md`](docs/specs/06-compatibility.md) | PySide6 6.6+ breaking changes, style conventions, testing strategy |

---

## Architecture Rules

- Each `core/` module is a **standalone pure-function module** — no GUI imports, no side effects. Takes pandas DataFrame in, returns DataFrame out.
- Each `visualization/` module returns a `matplotlib.figure.Figure` object. The GUI layer wraps it in `FigureCanvasQTAgg`.
- The `gui/` layer only handles layout, signals/slots, and user interaction. Zero processing logic in GUI code.
- The `pipeline.py` orchestrator chains core modules in MetaboAnalyst's exact order. It maintains a processing log (`list[str]`).
- All DataFrame operations must preserve the original index (sample IDs) and columns (feature names).
- Use `@staticmethod` for stateless transformations. Use classes only when state is needed (e.g., `MetaboAnalystPipeline`).

---

## Processing Pipeline Order (NEVER reorder)

```
Step 0: Zero → NaN conversion
Step 1: RemoveMissingPercent (threshold=0.5)
Step 2: ImputeMissingVar (default="min"/LoD)
Step 3: FilterVariable (default="iqr", auto-adaptive cutoff)
Step 4: Row-wise Normalization (sample-level correction)
Step 5: Data Transformation (USES GENERALIZED LOG, NOT STANDARD LOG)
Step 6: Column-wise Scaling (feature-level centering/scaling)
Step 7: Statistical analysis & visualization
```

**CRITICAL — Generalized Log Transform:** MetaboAnalyst's `"LogNorm"` is glog, NOT `log2(x+1)`. See `docs/specs/01-algorithms.md` Constraint 1 for formula and implementation.

---

## Project Structure

```
metaboanalyst_clone/
├── CLAUDE.md                  # This file (overview + rules)
├── docs/specs/                # Detailed specifications (6 files)
├── main.py                    # Entry point
├── requirements.txt
├── core/                      # Pure-function processing modules
│   ├── missing_values.py, filtering.py, normalization.py
│   ├── transformation.py, scaling.py, pipeline.py
├── analysis/                  # Statistical analysis
│   ├── pca.py, plsda.py, univariate.py, clustering.py
├── visualization/             # Returns matplotlib Figure objects
│   ├── pca_plot.py, volcano_plot.py, heatmap.py
│   ├── vip_plot.py, boxplot.py, density_plot.py
├── gui/                       # PySide6 GUI layer
│   ├── main_window.py, data_import_tab.py, missing_value_tab.py
│   ├── filter_tab.py, norm_tab.py, stats_tab.py, visual_tab.py
│   └── widgets/ (mpl_canvas.py, plotly_widget.py)
├── tests/                     # Unit + integration tests
├── translations/              # Qt .ts/.qm files
├── resources/                 # Fonts, etc.
├── assets/                    # Icons
└── deploy/                    # PyInstaller specs, Inno Setup
```

---

## Development Order

1. `core/missing_values.py` + tests
2. `core/filtering.py` + tests
3. `core/normalization.py` + tests
4. `core/transformation.py` + tests ← **validate glog carefully**
5. `core/scaling.py` + tests
6. `core/pipeline.py` + integration test
7. `analysis/pca.py` + `visualization/pca_plot.py`
8. `analysis/univariate.py` + `visualization/volcano_plot.py`
9. `analysis/plsda.py` + `visualization/vip_plot.py` ← **validate VIP carefully**
10. `analysis/clustering.py` + `visualization/heatmap.py`
11. `visualization/boxplot.py` + `visualization/density_plot.py`
12. `gui/widgets/` → `gui/main_window.py` → individual tab files
13. i18n: extract strings → translate zh_TW → compile .qm → integrate QTranslator
14. Deploy: PyInstaller spec → test builds → installer / code sign
15. `main.py` entry point with platform setup + language loading
16. CI/CD: GitHub Actions workflow

---

## Style & Convention

- Code language: **English** (variable names, docstrings, comments)
- GUI labels: **bilingual** — wrap in `self.tr()` for i18n
- Docstrings: Google style; type hints on all public functions
- `black` formatting, `isort` for imports, no wildcard imports
- Every `core/` function documents the corresponding MetaboAnalyst R function name

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Chao-hu-Lab)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Chao-hu-Lab)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
