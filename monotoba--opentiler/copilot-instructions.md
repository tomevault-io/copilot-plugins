## opentiler

> - `opentiler/`: Core app (Qt/PySide6 GUI, dialogs, exporter, settings, utils, viewer). Assets in `opentiler/assets/`.

 # Repository Guidelines

 ## Project Structure & Modules
 - `opentiler/`: Core app (Qt/PySide6 GUI, dialogs, exporter, settings, utils, viewer). Assets in `opentiler/assets/`.
 - `main.py`: CLI/GUI entry; calls `opentiler.main_app.main()`.
 - `tests/`: Pytest suite (unit, plugins, integration). Uses markers: `gui`, `integration`, `slow`.
 - `plugins/`: Plugin system (manager, hooks, builtins) and plugin docs.
 - `docs/`: User/dev docs and images. See `docs/developer/DEVELOPER_MANUAL.md`.
 - `tools/`: Utilities (screen capture, validation helpers).
 - Other: `requirements*.txt`, `setup.py`, `BUILD_INSTRUCTIONS.md`.

 ## Build, Test, Run
 - Install dev deps: `pip install -r requirements-dev.txt` (optional features via `requirements-optional.txt`).
 - Editable install: `pip install -e .` (adds `opentiler` / `opentiler-gui`).
 - Run app locally: `python main.py` or `opentiler`.
 - Test fast: `pytest -q`.
 - Test with markers: `pytest -m "not slow"`, `pytest -m gui`.
 - Coverage: `pytest --cov=opentiler --cov-report=term-missing`.
 - Build dist: `python -m build` or `python setup.py sdist bdist_wheel` (see `BUILD_INSTRUCTIONS.md`).

 ## Coding Style & Naming
 - Python 3.10+; 4-space indent; PEP 8.
 - Format: `black .`; Imports: `isort .`; Lint: `flake8`; Types: `mypy opentiler`.
 - Names: modules/functions `snake_case`, classes `CamelCase`, constants `UPPER_SNAKE`.
 - UI resources live under `opentiler/assets/`.

 ## Testing Guidelines
 - Framework: `pytest`; files `tests/test_*.py`; classes start with `Test*`; test funcs `test_*`.
 - GUI tests set `QT_QPA_PLATFORM=offscreen` (handled in `conftest.py`).
 - Use markers (`gui`, `integration`, `slow`) and fixtures (`qapp`, `temp_dir`, `mock_main_window`).
 - Aim for meaningful unit tests around utils, exporters, and plugins; add integration tests for workflows.

 ## Commit & Pull Requests
 - Commit style: short imperative subject with component scope, e.g. `print: set first-page orientation` or `fix(preview): import QSize`.
 - Include why when relevant; group related changes; keep diffs focused.
 - PRs: clear description, linked issues (e.g., `Closes #123`), steps to test, screenshots/GIFs for UI changes, and notes on risks/rollout.
 - CI hygiene: pass lint/format/type/tests locally before opening PR.

 ## Security & Configuration
 - Optional features: CAD (`ezdxf`), RAW (`rawpy`, `numpy`), automation (`mss`, `pywinctl`). Install only as needed.
 - Avoid committing large binaries; place assets under `opentiler/assets/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Monotoba)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Monotoba)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
