## python-style

> Python coding standards in ableton-cli


- Use `from __future__ import annotations` in new modules, mirroring src/ableton_cli/*.
- Format with ruff; do not introduce alternative tools.
- Raise `AppError` with explicit `error_code`/`exit_code` per `src/ableton_cli/errors.py`; do not catch and ignore.
- No fallback execution paths or compatibility shims.
- Tests live in `tests/` mirroring source layout; follow existing fixtures in `tests/conftest.py`.

---
> Source: [6uclz1/ableton-cli](https://github.com/6uclz1/ableton-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
