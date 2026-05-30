## vanguard-analysis

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Dependencies and environment are managed with `uv`. Never invoke `pip`, `python -m venv`, or activate `.venv/` manually.

```bash
uv sync                                 # install all deps (incl. dev) from uv.lock
uv sync --all-groups                    # same as CI; includes dev group
uv run pytest tests/                    # run full test suite
uv run pytest tests/test_analysis.py    # single test file
uv run pytest tests/test_analysis.py::test_name  # single test
uv run pytest --cov=vanguard_analysis --cov-report=term-missing  # with coverage
uv run jupyter notebook                 # open notebooks
uv run generate-notebooks               # regenerate ipynb files from _notebook_gen.py
uv run generate_notebooks.py            # equivalent standalone script
```

When adding dependencies use `uv add <pkg>` (or `uv add --group dev <pkg>`) so `pyproject.toml` and `uv.lock` stay in sync.

## Architecture

The package is a small ETL+analysis pipeline. Modules are deliberately separated by stage so each can be swapped or tested independently:

```
parser.py      -> accounts.py       -> enrichment.py     -> analysis.py
(CSV + yaml)     (per-account views)  (yfinance + txn      (classification +
                                       fallback yields)     tax-drag reports)
```

The canonical entry point is `vanguard_analysis.load(csv_path, config_path)`, which returns `(holdings, txns, config)` with account metadata already merged. Downstream code (`enrich_holdings`, `classify_holdings`, `rebalancing_report`, …) consumes these dataframes; everything is re-exported from `vanguard_analysis/__init__.py`.

### Vanguard CSV quirks

`parser.parse_csv` handles a non-obvious shape: the OFX export concatenates **two CSV tables in one file** — a holdings snapshot, then a blank line, then a transaction history with a different header. The parser locates the second header by searching for the string `"Trade Date"` and splits the file there. All column names are normalized to snake_case at parse time; `account_number` is coerced to `int` so merges with the account map are consistent.

### Account metadata

`data/accounts.yaml` (gitignored) maps account numbers to `label / type / owner / tax_advantaged`. `data/accounts-template.yaml` is the checked-in example. Account types in active use: `rollover_ira`, `roth_ira`, `traditional_ira`, `brokerage`, `cash`. `tax_advantaged: true` drives the tax-location logic in `analysis.py`.

### Tax-location classification

`analysis.classify_holdings` assigns each position one of: `dividend`, `growth`, `cash`, `muni`, `tax_managed`, `unknown`. Three hardcoded symbol sets at the top of `analysis.py` (`MUNI_FUNDS`, `TAX_MANAGED_FUNDS`, `CASH_FUNDS`) override the yield-based classification — these are intentional exceptions to the "dividend → IRA, growth → taxable" rule. When adding new exception funds, update those sets, not the classification logic.

`correctly_placed` is tri-state (`True`/`False`/`None`); `None` means cash or unknown-yield, which should never appear in misplacement reports.

### Yield enrichment with fallback

`enrichment.enrich_holdings` calls yfinance per unique symbol (cached via `lru_cache`). yfinance is an unofficial scraper and frequently returns nothing — when it does, and `txns` is passed, `derive_yield_from_transactions` computes a TTM yield from actual distribution history (sum of trailing-12-month dividend/capital-gain transactions ÷ current value). The `yield_fetch_status` column records which path was used (`ok` / `txn_derived` / `no_data` / `skipped`). Tests use the `enriched_holdings` fixture in `conftest.py` to bypass yfinance entirely.

### Notebooks are generated, not hand-edited

`notebooks/*.ipynb` are produced by `vanguard_analysis/_notebook_gen.py`. Edit the cell definitions in `_notebook_gen.py` and re-run `uv run generate-notebooks`; do not edit the `.ipynb` files directly (changes will be overwritten). The console-script entry point is wired in `pyproject.toml` via `_cli.py`.

## Testing notes

Tests build their dataframes from in-memory fixtures in `tests/conftest.py` (`SAMPLE_CSV`, `SAMPLE_CONFIG`) — no real CSV or network calls. The `enriched_holdings` fixture hand-sets `dividend_yield` values chosen to hit every branch in `classify_holdings` (growth, dividend, muni override, tax-managed override, cash override). When adding tests that involve enrichment, prefer this fixture pattern over mocking yfinance.

Coverage is configured in `pyproject.toml` to omit `_cli.py` and `_notebook_gen.py`.

---
> Source: [sulrich/vanguard-analysis](https://github.com/sulrich/vanguard-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
