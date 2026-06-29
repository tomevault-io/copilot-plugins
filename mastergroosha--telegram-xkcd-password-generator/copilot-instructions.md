## telegram-xkcd-password-generator

> - This project uses `uv` for Python environment and command execution.

# Project Instructions

- This project uses `uv` for Python environment and command execution.
- Use `uv run python ...` instead of plain `python`.
- Use `uv run ...` for project commands unless there is a specific reason not to.
- The project targets Python 3.14.
- Do not edit generated cache directories such as `__pycache__`.

## Running Tests

- Run all tests: `uv run pytest`
- Tests are located in the `tests/` directory.
- The test suite uses `pytest-asyncio` with `asyncio_mode = "auto"` (no need to decorate async tests manually).
- A shared `XKCDGenerator` fixture (session-scoped) is defined in `tests/conftest.py` and uses `words.txt` from the project root.

---
> Source: [MasterGroosha/telegram-xkcd-password-generator](https://github.com/MasterGroosha/telegram-xkcd-password-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
