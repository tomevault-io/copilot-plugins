## nbg-ynab-export

> This document gives contributor and coding-agent guidance for this repository.

# CLAUDE.md

This document gives contributor and coding-agent guidance for this repository.

## Quick Start

```bash
python3 -m venv venv
source venv/bin/activate  # Windows: .\\venv\\Scripts\\activate
pip install -r requirements.txt
npm install --save @actual-app/api
```

Runtime dependencies:
- `pandas`, `openpyxl`
- `PyQt5`
- `requests`
- `cryptography`

## Run Commands

CLI conversion:

```bash
python cli.py path/to/statement.[xlsx|xls|csv]
python cli.py path/to/statement.[xlsx|xls|csv] --previous path/to/previous_ynab.csv
```

GUI wizard:

```bash
python ui/wizard.py
```

Diagnostics (Actual API):

```bash
python scripts/actual_diag.py --url https://your-actual-host/api --password YOUR_PASSWORD
```

Quality checks:

```bash
pytest -q
flake8 .
```

## Architecture (Current)

- `converter/`
  - `account.py`: NBG account conversion
  - `card.py`: NBG card conversion
  - `revolut.py`: Revolut conversion (EUR-only, completed transactions only)
  - `dispatcher.py`: source detection by required-column subsets
  - `utils.py`: IO helpers, amount parsing, dedupe, CSV formula sanitization

- `services/`
  - `conversion_service.py`: canonical conversion entry points (`convert_to_ynab`, `convert_to_actual`)
  - `ynab_client.py`: YNAB HTTP client + API logging
  - `actual_client.py`: Actual API wrapper using Node bridge
  - `actual_bridge_runner.py`: process wrapper for `scripts/actual_bridge.js`
  - `token_manager.py`: Fernet key/token persistence

- `ui/`
  - `wizard.py`: stacked-page app shell and navigation logic
  - `controller.py`: threaded workers and async flow
  - `pages/`: import/auth/account/transactions/review/finish pages

## Critical Behavior Rules

1. De-duplication source of truth
- Use `converter.utils.exclude_existing` (re-exported via `services.conversion_service.exclude_existing_transactions`)
- Default behavior: remove exact duplicates only (`Date|Payee|Amount|Memo`)
- Legacy date-cutoff behavior exists behind `drop_older_than_latest_prev=True` and is not default CLI behavior

2. Detection + required columns
- Format detection happens in `converter/dispatcher.py` using required-column subsets from `constants.py`
- If required columns change, update:
  - `constants.py`
  - `converter/dispatcher.py`
  - converter logic/tests relying on those headers

3. Secret handling
- Do not reimplement encryption in UI pages
- Use `services/token_manager.py`
- Preserve `0600` permission behavior on settings/key/log files

4. CSV safety
- Preserve formula escaping via `sanitize_csv_formulas` before writing CSV outputs

5. Actual bridge behavior
- Bridge contract is JSON-lines over stdio (`scripts/actual_bridge.js`)
- Keep field mapping stable (`amount` milliunits in Python side; bridge converts for Actual)
- Preserve retry path for out-of-sync migrations in `ActualClient`

## UI Flow Notes

Export targets:
- `YNAB`
- `ACTUAL_API`
- `FILE`

Flow differences:
- `ACTUAL_API` routes authorize step to `ActualAuthPage`
- `FILE` mode skips auth/account/transactions and jumps Import -> Review -> Finish

Duplicate review behavior:
- Duplicates are pre-marked as skipped
- User can explicitly unskip and upload anyway

## Testing Expectations

- Keep all existing tests passing (`pytest -q` currently covers 200 tests)
- Add tests for any behavior change in converters/services/UI workers
- Do not delete tests to satisfy changes

Focus test files by area:
- converter/service logic: `tests/test_conversion_service.py`, `tests/test_utils.py`, `tests/test_dispatcher.py`
- API clients/bridge: `tests/test_ynab_client.py`, `tests/test_actual_client.py`, `tests/test_actual_bridge_runner.py`
- UI behavior: `tests/test_ui_pages.py`, `tests/test_ui_controller.py`, `tests/test_wizard_workflow.py`

## Style and Conventions

- Python, 4-space indent, max line length 120 (`.flake8`)
- Keep entry points (`cli.py`, `ui/wizard.py`) thin; put logic into `services/` and `converter/`
- Use small functions and explicit errors when data is invalid

## Git / PR Guidance

- Commit messages: short, imperative (`fix:`, `refactor:`, `UI:`, `test:` optional)
- PRs should include:
  - behavior summary
  - rationale
  - test evidence (`pytest`, `flake8`)
  - UI screenshots for visual changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nikolay-Sergeev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
