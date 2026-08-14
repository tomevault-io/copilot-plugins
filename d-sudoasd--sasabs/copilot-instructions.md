## sasabs

> `src/saxsabs/` contains the installable Python package. Core scientific logic lives in `src/saxsabs/core/`, file parsing and exporters in `src/saxsabs/io/`, and command-line wiring in `src/saxsabs/cli.py` plus `src/saxsabs/__main__.py`. Root-level `SASAbs.py`, `saxsabs_workbench.py`, `saxsabs_workbench.pyw`, and `Start_SAXSAbs_Workbench.bat` support the legacy/desktop workbench. Tests are in `tests/`. Example inputs and manual workflow checks are in `examples/`; architecture and reviewer documentation are in `docs/`; paper and submission assets are under `paper/` and `submission/`.

# Repository Guidelines

## Project Structure & Module Organization

`src/saxsabs/` contains the installable Python package. Core scientific logic lives in `src/saxsabs/core/`, file parsing and exporters in `src/saxsabs/io/`, and command-line wiring in `src/saxsabs/cli.py` plus `src/saxsabs/__main__.py`. Root-level `SASAbs.py`, `saxsabs_workbench.py`, `saxsabs_workbench.pyw`, and `Start_SAXSAbs_Workbench.bat` support the legacy/desktop workbench. Tests are in `tests/`. Example inputs and manual workflow checks are in `examples/`; architecture and reviewer documentation are in `docs/`; paper and submission assets are under `paper/` and `submission/`.

## Build, Test, and Development Commands

- `pip install -e .[dev]`: install the package in editable mode with pytest, ruff, and plotting test dependencies.
- `pip install -e .[gui,hdf5]`: install optional GUI, detector-image, and HDF5 support when working on workbench or NXcanSAS paths.
- `pytest -q`: run the automated test suite configured by `pyproject.toml`.
- `ruff check src tests`: run static lint checks with the repository's 100-character line limit.
- `python -m saxsabs --version` or `saxsabs --version`: verify the CLI entry point after installation.
- `python saxsabs_workbench.py --lang en`: launch the desktop workbench locally.

## Coding Style & Naming Conventions

Use Python 3.10+ and follow the existing style: 4-space indentation, explicit imports, small pure functions for scientific calculations, and type hints where they clarify public or shared behavior. Keep package code inside `src/saxsabs/`; avoid adding new root-level modules unless they are launchers or compatibility shims. Name tests `test_*.py`, test functions `test_*`, and test classes `Test*`. Keep GUI orchestration separate from reusable core logic.

## Testing Guidelines

Add or update focused pytest coverage for behavior changes, especially validation, numerical calculations, parsers, and CLI output. Use `tmp_path`, `capsys`, and `monkeypatch` patterns already present in `tests/`. For GUI or end-to-end workflow changes, also check `examples/manual-verification.md` and the minimal 2D package in `examples/minimal_2d/`.

## Commit & Pull Request Guidelines

Recent history uses short subjects, often typed prefixes such as `fix:`, `test:`, and `style:`. Prefer `type: imperative summary` for new commits, for example `fix: reject invalid transmission values`. Pull requests should describe the changed workflow or API, link relevant issues, include tests or manual verification notes, and update docs when CLI, GUI, or public API behavior changes.

## Security & Configuration Tips

Do not commit beamline-private datasets, credentials, or large generated outputs. Keep examples anonymized and small. Optional dependencies should remain optional unless the core CLI/API truly requires them.

---
> Source: [D-sudoasd/SASAbs](https://github.com/D-sudoasd/SASAbs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
