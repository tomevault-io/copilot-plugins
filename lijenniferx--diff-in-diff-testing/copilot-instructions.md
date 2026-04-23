## diff-in-diff-testing

> This project is for learning and exploring difference-in-differences (DiD) estimators, including:

# diff_in_diff Project

## Overview
This project is for learning and exploring difference-in-differences (DiD) estimators, including:
- **Standard 2x2 DiD** — two periods, two groups, clean parallel trends
- **Honest DiD** — sensitivity analysis when parallel trends may be violated
- **Callaway-Sant'Anna** — staggered adoption with heterogeneous treatment timing

Primary language: Python (Google Colab notebooks)

---

## Environment

- Runtime: Google Colab (Python 3.x)
- State resets on runtime reconnect — re-run setup cells after reconnecting
- Install packages with `!pip install <package>`

Key libraries assumed available:
- `numpy`, `pandas`, `scipy`
- `matplotlib`, `seaborn`
- `scikit-learn`
- `statsmodels`

---

## Helper Functions — CRITICAL RULE

**Before writing any utility code, ALWAYS read the utils/ files first:**

- `utils/data.py` — synthetic data generation (panel data, violated trends, staggered adoption)
- `utils/plotting.py` — visualization helpers (parallel trends, event study, cohort trends)
- `utils/stats.py` — summary utilities and estimator wrappers

**If a function could be reused across notebooks, add it to the appropriate utils module instead of defining it inline.**

To import in a notebook:
```python
import sys
sys.path.insert(0, '/content/drive/MyDrive/Colab Notebooks/diff_in_diff_testing')

from utils.data import make_clean_panel, make_violated_trends_panel, make_staggered_panel
from utils.plotting import plot_parallel_trends, plot_event_study, plot_cohort_trends
from utils.stats import summarize_panel
```

---

## Code Conventions

- One logical step per cell; use markdown cells as section headers
- Each notebook must have a markdown cell at the top with: title, purpose, and date
- Use descriptive variable names (no single-letter vars except loop indices)
- Always set a random seed: `np.random.seed(42)` or pass `seed=42` to data generators
- Prefer `pandas`/`numpy` over raw Python loops for data operations
- Use f-strings for string formatting
- Show intermediate results with `display(df.head())` or `print(df.shape)`

---

## Visualization Conventions

- Always label axes and add a title
- Call `plt.tight_layout()` before `plt.show()`
- Prefer `seaborn` for statistical plots, `matplotlib` for custom plots

---

## Notebook Organization

- One notebook per estimator or concept
- Name notebooks descriptively: `01_standard_did.ipynb`, `02_honest_did.ipynb`, `03_callaway_santanna.ipynb`
- State the learning objective clearly in the top markdown cell

---

## Code Review — REQUIRED AFTER EVERY CHANGE

**After making any edit to a file in this project, you MUST spawn a reviewer subagent before
presenting results to the user.**

Use the Agent tool with `subagent_type: "general-purpose"` and instruct it to:

1. Read every file that was just modified
2. Check for the following and report findings:
   - **Correctness**: does the logic match what was intended?
   - **Duplication**: does any new function already exist in utils/?
   - **API consistency**: do wrappers match the diff-diff package signatures in `diff-diff/diff_diff/`?
   - **Column name alignment**: do default column names match those produced by `utils/data.py`
     (`unit_id`, `period`, `treated`, `post`, `outcome`, `g`, `ever_treated`, `treated_at_t`)?
   - **Docstrings**: are parameters and return values described?
   - **Style violations**: single-letter variable names (except loop indices), missing f-strings,
     raw Python loops where pandas/numpy would be cleaner

3. Return a short findings report: list issues found (if any) or confirm the change looks good.

If the reviewer finds issues, fix them before responding to the user.

---

## What to Avoid

- Do not define utility functions inline in notebooks if they belong in utils/
- Do not overengineer — this is exploratory code, not production
- Do not add error handling for scenarios that cannot happen in a notebook context
- Do not duplicate functions already in utils/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lijenniferx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
