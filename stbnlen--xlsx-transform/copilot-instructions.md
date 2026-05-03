## xlsx-transform

> A Streamlit application for uploading, transforming, analyzing, and downloading Excel files. It includes data filtering modules (Q_BANCO, Q_CMR), payment analysis (PAGOS_FRM, PAGOS BCI), assignment processing (FORUM, BCI), and a contact center analytics module (New CD).

# Agent Guidelines for xlsx-transform

## Project Overview

A Streamlit application for uploading, transforming, analyzing, and downloading Excel files. It includes data filtering modules (Q_BANCO, Q_CMR), payment analysis (PAGOS_FRM, PAGOS BCI), assignment processing (FORUM, BCI), and a contact center analytics module (New CD).

## Commands

### Running the Application

```bash
streamlit run app.py
```

### Dependencies

```bash
pip install -r requirements.txt
```

### Testing

```bash
pip install pytest
pytest                        # Run all tests
pytest tests/                 # Run tests in specific directory
pytest tests/test_file.py::test_function_name  # Run single test
```

### Linting and Formatting

```bash
pip install ruff black isort
ruff check .                  # Lint all files
ruff check --fix .            # Auto-fix issues
black .                       # Format with black (line length 88)
isort --profile=black .       # Sort imports
```

Full formatting pipeline:

```bash
isort --profile=black . && black .
```

## Code Style Guidelines

### General

- Python 3.x compatible code
- 4 spaces for indentation (no tabs)
- Line length: 88 characters (black default)
- Double quotes for strings (enforced by black)
- Add type hints to all function parameters and return values

### Language

- **All code variables, function names, and internal DataFrame columns**: English
- **Source Excel column names** (matching uploaded data): keep original Spanish names in matching logic
- **UI strings** (labels, messages, chart titles): English
- **Business domain terms** in data (RUT, Castigo, Vigente, etc.): keep as-is in data context

### Imports

```python
# Standard library first
import io
import re

# Third-party libraries
import numpy as np
import pandas as pd
import streamlit as st

# Local imports
from utils import aggregate_monthly, process_date_columns
```

- Use `isort --profile=black` for consistent import ordering
- Never use `sys.path.append` hacks

### Naming Conventions

- **Functions/variables**: `snake_case` (e.g., `process_data`, `monthly_df`)
- **Classes**: `PascalCase` (e.g., `DataProcessor`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `MONTH_LABELS`)
- **Private helpers**: prefix with underscore (e.g., `_show_trend_analysis`)
- **DataFrame columns** created by our code: English `snake_case` (e.g., `total_amount`, `payment_count`, `days_in_month`)
- **Source columns** from Excel: preserve original names for data matching

### Type Annotations

All public functions must have type hints:

```python
def process_file(file: io.BytesIO) -> pd.DataFrame:
    """Process uploaded Excel file and return DataFrame."""
    return pd.read_excel(file)
```

### Error Handling

- Always use `except Exception:` (never bare `except:`)
- Provide meaningful error messages
- Use `st.error()` for user-facing errors

```python
try:
    df = pd.read_excel(uploaded_file)
except Exception as e:
    st.error(f"Error reading file: {e}")
```

### Streamlit Patterns

- `st.file_uploader` for file inputs with `type=["xlsx", "xls"]`
- `st.dataframe` for table previews
- `st.download_button` always use `output.getvalue()` (not the BytesIO object directly)
- `io.BytesIO` for in-memory file operations
- No `print()` statements in production code

### File Structure

```
xlsx-transform/
├── app.py              # Main Streamlit entry point
├── utils.py            # Shared utility functions (stats, charts)
├── utils_new_cd.py     # Utilities for New CD module
├── q_banco.py          # Q_BANCO view module
├── q_cmr.py            # Q_CMR view module
├── pagos_frm.py        # PAGOS_FRM view module (analysis + ML)
├── pagos_bci.py        # PAGOS BCI and BCI merge views
├── pages/
│   ├── asig.py         # Asignaciones page (tabs: Q_BANCO, Q_CMR, FORUM, BCI)
│   ├── pagos.py        # Payments page (tabs: PAGOS_FRM, PAGOS BCI)
│   └── new_cd.py       # New CD analytics page
├── tests/
│   ├── test_app.py
│   ├── test_utils.py
│   ├── test_q_banco.py
│   ├── test_q_cmr.py
│   ├── test_pagos_bci.py
│   ├── test_pagos_frm.py
│   └── test_new_cd.py
└── requirements.txt
```

### Key DataFrame Column Names (Internal)

These are the English column names used in DataFrames created by the utility functions:

| Function | Column Name | Description |
|---|---|---|
| `process_date_columns` | `year`, `month_num`, `day`, `day_of_week`, `year_month` | Date features |
| `aggregate_monthly` | `total_amount`, `payment_count`, `avg_amount`, `median_amount`, `std_amount`, `days_in_month` | Monthly aggregates |
| `calculate_yearly_stats` | `growth`, `executives_count` | Yearly stats |
| `calculate_seasonal_indices` | `Month`, `Seasonal Index` | Seasonal analysis |
| `create_features` (utils_new_cd) | `day_of_week`, `day_of_month`, `month`, `year`, `is_monday`, `is_friday`, `is_weekend`, `week_of_month`, `fortnight`, etc. | Calendar features |
| `train_and_predict` | `date`, `day_of_month`, `weekday_num`, `mandante_name`, `weekday_name`, `prediction` | Prediction output |

### Security Considerations

- Never hardcode secrets or API keys
- Use Streamlit session state for user-specific data
- Validate file types before processing

### Best Practices

1. Run `isort --profile=black . && black .` before committing
2. Run `ruff check .` and fix any issues before committing
3. Run `pytest` before pushing to verify no regressions
4. Use `io.BytesIO` for in-memory file operations
5. Add docstrings to all functions
6. Use descriptive English variable names
7. Keep business domain terms (RUT, Castigo, etc.) as-is in data contexts

---
> Source: [stbnlen/xlsx-transform](https://github.com/stbnlen/xlsx-transform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
