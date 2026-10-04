## python-conventions-mdc

> Python style and typing conventions for CursorFleet source and tests


- Python >=3.11, fully typed (`mypy --strict`), formatted/linted with ruff (line length 100).
- Use `from __future__ import annotations` in new modules.
- Subprocess calls: argv list, explicit `timeout=`, never `shell=True`.
- Tests: no `time.sleep`; prefer deterministic fakes, `tmp_path`, and Textual `run_test()` for TUI.
- Tests live in `tests/{unit,integration,tui,security}`; fixtures in `tests/fixtures/cursor` must be sanitized.
- CLI command groups: one module per group in `src/cursorfleet/cli/commands/` exposing `register(app)`.

---
> Source: [M9nx/cursorfleet](https://github.com/M9nx/cursorfleet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
