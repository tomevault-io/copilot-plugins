## broadcast-tracking

> Chess Tournament Digital Board Tracker — PyQt6 / SQLAlchemy / SQLite.

Chess Tournament Digital Board Tracker — PyQt6 / SQLAlchemy / SQLite.

## Architecture

- ALL SQLAlchemy queries MUST go through `database/queries.py`. No `session.query()` in logic or presenters.
- Presenters in `gui/presenters/` are lazy-loaded (`@property` + `@setter`). Mock via `@setter`.
- Export: `export(session, id, path, format_type)` dispatcher. Keys: `'CSV'`, `'JSON'`, `'Statistics'` (case-sensitive).
- Public APIs re-exported from `__init__.py` in `database/`, `logic/`, `utils/`, `scrapers/`, `gui/presenters/`.

## Scrapers

URL-based routing in `ScraperPresenter.fetch_and_import()`: `chess-results.com` → `ChessResultsScraper`; `member.schack.se` → `SchackSeApiScraper` (HTML fallback); unknown → Schack.se. Schack.se HTML broken for individual tournaments (JS-loaded links).

## Style

Built-in generics (`list[X]`, `X | None`). `Callable` from `collections.abc`. No bare `except`, no `raise e`, no inline imports. Magic numbers in `config.py` or class constants.

## Testing

`pytest tests/` (324 tests, 96%). Skip GUI with `-k "not gui"`. Shared fixtures in `conftest.py`. Patch `SchackSeScraper` on presenter module; inline scrapers via `patch.dict('scrapers.__dict__', ...)`. `CsvExporter` requires `digital_assignment`. `AllocationPresenter.remove_pairing` needs `QMessageBox.question` mocked to `Yes`. `PairingCardBuilder` tests need `QApplication`. Flaky: `test_manual_entry_preserves_on_reallocation`.

## Pitfalls

- `Column[bool]` always truthy — use `Column == False`.
- `session.flush()` after `session.delete()` before INSERT (UNIQUE error).
- `edit_pairing` creates `Participant` without flush → IntegrityError.
- `get_pairing_digital_sum` returns `int | None`.
- Parse "½" → ".5" suffix (NOT "0.5").
- `get_tournament` returns `Tournament | None` — guard before `.name`.
- `ManualPairingDialog` uses QComboBox `.currentText()`.
- `SchackSeScraper` Method 1 needs `listheader` + `listheadercenter` + 2+ rows.

---
> Source: [Claes1981/broadcast-tracking](https://github.com/Claes1981/broadcast-tracking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
