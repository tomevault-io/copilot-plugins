## pytest-cocotb

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## IMPORTANT: Always activate the venv and use uv for package management

ALL Python and pytest commands MUST be run with the project venv activated:

```
source /home/sjalloq/Work/cocotb-pytest/.venv/bin/activate
```

Do this BEFORE running any python or pytest commands. Never use bare system python.

Always use `uv` instead of `pip` for installing packages (e.g., `uv pip install` not `pip install`).

## Commands

```bash
# Install in development mode
uv pip install -e .

# Run all unit tests (no simulator needed)
pytest tests/test_plugin.py

# Run a single test class or test
pytest tests/test_plugin.py::TestTestSession
pytest tests/test_plugin.py::TestTestSession::test_run_delegates_to_runner

# Run e2e tests (requires verilator on PATH)
pytest tests/e2e/test_counter.py --sim verilator --hdl-toplevel counter --sources tests/e2e/rtl/counter.sv

# Run the full test suite (simulator-dependent tests auto-skip if verilator missing)
pytest
```

## Architecture

This is a **pytest plugin** (`pytest-cocotb`) that integrates cocotb HDL simulation into pytest. It is registered via the `pytest11` entry point in `pyproject.toml`.

### Key components

- **`src/pytest_cocotb/plugin.py`** — The pytest plugin entry point. Registers CLI options (`--sim`, `--hdl-toplevel`, `--sources`, `--parameters`, `--waves`, `--clean`, etc.) and provides fixtures:
  - `runner` (session-scoped) — Compiles HDL once per pytest session using `cocotb_tools.runner.get_runner()`
  - `test_session` (function-scoped) — Yields a `TestSession` bound to a unique per-test directory
  - `build_dir`, `sim_build_dir`, `testrun_uid` — Directory management fixtures

- **`src/pytest_cocotb/session.py`** — `TestSession` dataclass that wraps `runner.test()` with defaults and a single-use guard (can only call `run()` once per test).

### How a test works

A pytest test function receives the `test_session` fixture and calls `test_session.run(test_module="cocotb_module")`. This invokes the simulator on already-compiled HDL, running the cocotb coroutine tests defined in that module. The cocotb test module (e.g., `cocotb_counter.py`) is a separate file with `@cocotb.test()` decorated async functions — it is NOT collected by pytest directly.

### Test structure

- `tests/test_plugin.py` — Unit tests for `TestSession` and `_sanitise_name`, plus pytester-based integration tests for CLI options and rebuild behavior. Uses `pytester` plugin (registered in `tests/conftest.py`). Simulator-dependent tests use the `_needs_verilator` fixture to auto-skip.
- `tests/e2e/` — End-to-end test with real RTL (`counter.sv`) and cocotb testbench (`cocotb_counter.py`).

### Build system

Uses `hatchling` as the build backend. The package source lives under `src/pytest_cocotb/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sjalloq)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sjalloq)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
