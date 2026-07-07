## streamlit-dcf

> This project is a simplified Discounted Cash Flow (DCF) valuation model intended for educational purposes. The goal is **not** to produce the most complex valuation model possible, but rather to create one that is:

# CLAUDE.md

## Project Overview

This project is a simplified Discounted Cash Flow (DCF) valuation model intended for educational purposes. The goal is **not** to produce the most complex valuation model possible, but rather to create one that is:

- Easy to understand
- Easy to modify
- Transparent
- Academically defensible
- Suitable for individual investors

The philosophy of this project is that **all DCF models are highly sensitive to assumptions**, so simplicity and transparency are often more valuable than adding dozens of inputs that create an illusion of precision.

---

# Primary Goals

The application should:

- Allow users to upload financial statement data (Excel files exported from Stock Analysis or similar providers).
- Automatically calculate historical metrics.
- Project future Free Cash Flow.
- Estimate intrinsic value using a Discounted Cash Flow model.
- Allow assumptions to be easily modified.
- Display intermediate calculations whenever practical.

The project is educational first and foremost.

---

# Design Philosophy

When making implementation decisions, prefer:

1. Readability over cleverness.
2. Simplicity over excessive flexibility.
3. Explicit calculations over "magic."
4. Pure functions whenever possible.
5. Small modules with clear responsibilities.
6. Conservative assumptions.

Avoid unnecessary abstraction until it is actually needed.

---

# DCF Philosophy

This project intentionally follows a conservative valuation methodology.

## Historical Growth

The default projection method uses:

- 5-year historical CAGR
- Applied to Years 1–5

This default is editable: the UI pre-fills the Years 1-5 growth rate with the
calculated CAGR, but the user may override it directly.

Future versions may optionally allow:

- Analyst estimates
- Scenario analysis

However, historical CAGR remains the default because it is objective, reproducible, and easy to explain.

---

## Growth Deceleration

Growth should gradually slow over time.

The default implementation projects:

Years 1–5:
- Historical CAGR

Years 6–10:
- Linear transition toward Terminal Growth Rate

This avoids unrealistic perpetual high growth.

The Years 6-10 fade is defined by two editable bounds — a Year 6 (fade start)
rate and a Year 10 (fade end) rate — both pre-filled so the default schedule
transitions smoothly to the Terminal Growth Rate. The user may override either
bound independently, including collapsing the fade entirely (setting Years 1-5,
Year 6, and Year 10 to the same rate). An overridden Year 10 rate is not forced
to match the Terminal Growth Rate used in the terminal value calculation —
transparency and user control take priority over enforcing that consistency
automatically.

---

## Terminal Growth Rate

The default terminal growth rate is:

**2%**

Reasoning:

- Conservative
- Easy to defend
- Consistent with mature businesses
- Avoids unrealistic perpetual growth assumptions

Users may change this assumption.

---

## Discount Rate

The application should support:

- Manual WACC entry
- Optional calculated WACC in future versions

The user should always be able to override calculated values.

---

## Transparency

Users should always be able to answer:

"Where did this number come from?"

Avoid hidden calculations.

Whenever practical:

- Display formulas.
- Show intermediate values.
- Label assumptions clearly.

---

# Coding Standards

## Language

Python 3.12+

---

## Style

Follow PEP8.

Use:

- dataclasses
- pathlib
- type hints
- docstrings
- descriptive variable names

Avoid:

- global variables
- deeply nested logic
- unnecessary inheritance

---

## Functions

Functions should generally do one thing.

Prefer:

```python
calculate_cagr()

calculate_fcf_margin()

discount_cash_flow()

calculate_terminal_value()
```

over giant multi-purpose functions.

The valuation engine (`run_dcf()` in `dcf.py`) is the one place allowed to orchestrate
multiple steps (building the growth schedule, projecting revenue/FCF, discounting,
and computing terminal value), since a DCF run is inherently a single pipeline. It
should still delegate the underlying math to the single-purpose functions in
`calculations.py` rather than reimplementing them inline.

---

## Constants

Avoid magic numbers.

Default assumptions are expressed as dataclass field defaults (or Streamlit widget
defaults for user-facing inputs), not standalone module-level constants.

Example:

```python
@dataclass
class DCFInputs:
    ...
    forecast_years: int = 10
```

---

## Error Handling

Never silently ignore errors.

Prefer:

- informative exceptions
- user-friendly validation messages

---

# Project Structure

The project is intentionally flat — a handful of top-level modules rather than a
nested package:

```
DCF-App/

    calculations.py          # Pure financial math: CAGR, FCF, FCF margin,
                              # discounting, terminal value, shares outstanding
    dcf.py                   # Valuation engine: DCFInputs dataclass + run_dcf()
    loader.py                # Excel parsing: FinancialData dataclass +
                              # load_financial_data()
    main.py                  # Streamlit UI / application entry point
    dcf_test.py               # Manual verification script run against a sample
                              # workbook (not a pytest suite)

    data_format_template/    # Example StockAnalysis workbook layout
    pyproject.toml
    requirements.txt
    uv.lock
```

As the project grows, calculations, models, and services may be split into
subpackages — but only once the flat structure actually becomes hard to navigate.

---

# User Experience

The target audience is:

- Individual investors
- Students
- YouTube viewers
- Beginning value investors

The interface should favor:

- clarity
- explanation
- discoverability

over maximizing the number of options.

---

# Future Roadmap

Potential future enhancements:

- Analyst estimate import
- Multiple valuation scenarios
- Sensitivity analysis
- Monte Carlo simulation
- Reverse DCF
- Comparable company valuation
- Historical valuation charts
- AI-generated valuation summaries

These should not complicate the initial release.

---

# Testing Philosophy

Whenever calculations are added:

- include unit tests
- verify against Excel examples
- verify against hand calculations where practical

Financial calculations should prioritize correctness over optimization.

---

# AI Assistant Instructions

When generating code:

- Keep functions small.
- Prefer readability over clever optimization.
- Explain non-obvious financial calculations.
- Avoid unnecessary frameworks.
- Minimize dependencies.
- Preserve backward compatibility when practical.

If there are multiple reasonable implementation options, prefer the simplest solution that satisfies the requirements.

---

# Core Principle

> "A valuation model is only as good as its assumptions. Our job is not to predict the future with certainty, but to build a transparent framework that allows reasonable assumptions to be tested."

---
> Source: [techhound/streamlit-dcf](https://github.com/techhound/streamlit-dcf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
