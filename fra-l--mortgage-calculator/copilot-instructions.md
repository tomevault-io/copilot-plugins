## mortgage-calculator

> The file `.claude/memory/MEMORY.md` in this repository contains the persistent

# Claude Code — Project Context

## Auto-memory
The file `.claude/memory/MEMORY.md` in this repository contains the persistent
project memory (architecture, issue status, key patterns, test notes).

**On a new machine:** copy it to the Claude Code memory directory so it is
loaded automatically on every session:

```bash
mkdir -p ~/.claude/projects/-home-franc-mortgage-calculator/memory
cp .claude/memory/MEMORY.md ~/.claude/projects/-home-franc-mortgage-calculator/memory/MEMORY.md
```

If the project is cloned to a different path, adjust the encoded directory name
(replace `/` with `-` and drop the leading `/`, e.g. `/home/alice/mortgage-calculator`
→ `-home-alice-mortgage-calculator`).

---

<!-- The content below mirrors .claude/memory/MEMORY.md and is kept in sync. -->

## Project Overview
Danish mortgage analysis tool with PyQt6 GUI, matplotlib charts, cross-border foreign property analysis.

## Key Files
- `src/mortgage_calculator/gui.py` — Main GUI (QMainWindow + all tab widgets)
- `src/mortgage_calculator/calculator.py` — Core mortgage math
- `src/mortgage_calculator/comparison.py` — Institution comparison + breakeven
- `src/mortgage_calculator/models.py` — Pydantic models (LoanParams, LoanResult, ForeignPropertyParams, etc.)
- `src/mortgage_calculator/tax.py` — Rentefradrag + foreign property cross-border tax logic
- `src/mortgage_calculator/data/rates.py` — Hardcoded rate data (Feb 2026)
- `src/mortgage_calculator/cli.py` — Rich CLI (8-step flow)

## GUI Tab Layout
- Tab 0: Comparison (`ComparisonTableWidget`)
- Tab 1: Amortization (`AmortizationChartWidget`) — matplotlib waterfall
- Tab 2: Payment Breakdown (`PaymentBreakdownChartWidget`) — annual stacked bars
- Tab 3: Cost Comparison (`CostComparisonWidget`) — cumulative lines + pie
- Tab 4: Tax & Costs (`TaxCostsPanelWidget`) — QGroupBox panels
- Tab 5: Foreign Property (`ForeignPropertyPanelWidget`) — self-contained input + P&L + debt ceiling

## CI Workflow
- `.github/workflows/tests.yml` — pytest with `QT_QPA_PLATFORM=offscreen`
- Requires `libegl1 libgl1` apt packages for PyQt6 on Ubuntu runner (added in PR #22)

## Issue Status
- #16 ✅ Amortization & payment breakdown charts (merged PR #22)
- #17 ✅ Institution comparison & cost breakdown charts (merged PR #23)
- #18 ✅ Tax & one-time costs panels (merged PR #24)
- #19 ✅ Foreign property tab — implemented as general-purpose (PR for issue #26)
- #20 ✅ Export & final polish (merged PR #25)
- #26 ✅ General-purpose foreign property logic (replaces Italian-specific model)

## Key Patterns
- Each tab is a separate QWidget class with a `refresh(...)` or `set_loan_result(...)` method
- All chart widgets use `Figure(constrained_layout=True)` + `FigureCanvasQTAgg`
- `_update_tabs()` in MortgageWindow calls all tab refreshes after each compute
- `InputPanel` emits `params_ready(LoanParams)` (valid) and `params_invalid(str)` (error)
- Export button disabled until first successful computation
- `ForeignPropertyPanelWidget` is self-contained (has its own input form + Compute button)
- Cross-border tax: credit method — foreign tax paid first, DK top-up = (DK_rate − foreign_rate) × base

## Test Notes
- GUI tests need `QApplication.instance() or QApplication([])` fixture
- `io_years` spinbox is capped at 29 (cannot equal `term_years=30` to trigger error)
- Use `prop_value.setValue(100_000)` to reliably trigger LTV>80% validation error

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fra-l)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fra-l)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
