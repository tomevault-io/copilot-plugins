## tkinter-auto-clicker

> The core package is `autoclicker/`:

# Repository Guidelines

## Project Structure & Module Organization
The core package is `autoclicker/`:
- `__main__.py` boots DPI setup and launches the Tkinter app.
- `ui.py` contains the GUI (`AutoClickerApp`) and user interactions.
- `click_engine.py` handles click injection and the worker thread.
- `monitors.py` and `models.py` hold monitor logic and dataclasses.
- `hotkey.py` manages global F8 handling; `dpi.py` configures Windows DPI awareness.

Tests live in `tests/` (`test_coordinates.py`, `test_worker.py`, `test_click_engine.py`). Packaging/build metadata is in `pyproject.toml`, with executable build config in `windows_autoclicker.spec` and `build.bat`.

## Build, Test, and Development Commands
- `python -m venv .venv` then activate (`.venv\Scripts\activate` on Windows).
- `pip install -e .` installs the package in editable mode.
- `python -m autoclicker` runs the GUI locally.
- `pytest` or `pytest tests/` runs all tests.
- `build.bat` builds `dist/WindowsAutoClicker.exe` via PyInstaller (Windows).
- `pyinstaller windows_autoclicker.spec --clean` is the manual build equivalent.

## Coding Style & Naming Conventions
Use Python 3.8+ with 4-space indentation and type hints on new/changed code. Follow existing naming:
- `PascalCase` for classes
- `snake_case` for functions/variables
- `UPPER_CASE` for constants
- leading underscore for internal helpers

Prefer small, focused functions and explicit error paths. Keep Windows-specific API calls isolated to engine/DPI modules; keep UI thread responsive by offloading work to worker threads.

## Testing Guidelines
Use `pytest` and keep tests deterministic. Name files `test_*.py` and test functions `test_*`. Add tests for:
- normal behavior
- edge cases (bounds, timing, thread stop behavior)
- failure modes (Windows API errors, invalid coordinates)

For bugs, add a regression test first, then implement the fix.

## Commit & Pull Request Guidelines
Current history uses concise, imperative commit subjects, with occasional Conventional Commit prefixes (for example, `feat: ...`). Preferred format:
- `feat: add monitor refresh debounce`
- `Fix worker shutdown race`

PRs should include:
- what changed and why
- test evidence (`pytest` output)
- screenshots/GIFs for GUI changes
- linked issue/context when applicable

## Security & Configuration Tips
This app injects mouse input; validate coordinates and interval inputs at boundaries. Do not hardcode secrets or machine-specific paths. Test auto-click behavior in a safe environment before releasing binaries.

---
> Source: [AnhTrieu/tkinter-auto-clicker](https://github.com/AnhTrieu/tkinter-auto-clicker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
