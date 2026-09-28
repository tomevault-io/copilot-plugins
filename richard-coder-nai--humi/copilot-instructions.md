## humi

> Core code lives under `src/humi_deploy/`. Keep new modules inside the

# Repository Guidelines

## Project Structure & Module Organization
Core code lives under `src/humi_deploy/`. Keep new modules inside the
existing domains: `camera/` for device input, `data_recording/` for capture
pipelines, `robot/g1/` for robot-specific logic, `shared_memory/` for IPC,
and `common/` for reusable utilities. Tests live under `tests/` and mirror
the package layout, for example `tests/robot/g1/test_interface.py`. Use
`scripts/` for local debugging helpers such as camera or recorder probes.
Generated data belongs in `data/`; it is ignored by Git.

## Build, Test, and Development Commands
This project targets Python 3.11 and uses `uv`.

- `uv sync --dev`: install runtime and development dependencies from
  `pyproject.toml` and `uv.lock`.
- `make format`: run `docformatter`, `ruff format`, and `ruff check --fix`.
- `make type`: run `pyright` against the source tree.
- `make test`: run `pytest` with plugin autoload disabled for reproducibility.
- `uv run python scripts/debug_camera.py`: example local debug entry point.

Run `make format type test` before opening a pull request.

## Coding Style & Naming Conventions
Follow the current Python style: 4-space indentation, type hints on public
APIs, dataclasses where they simplify structured data, and concise docstrings
wrapped to 79 columns. Ruff is the formatter and linter; import order is also
enforced by Ruff. Use `snake_case` for functions, modules, and variables,
`PascalCase` for classes, and `UPPER_CASE` for constants. Prefer package paths
that stay aligned with domain boundaries, for example
`humi_deploy.robot.g1.interface`.
Whenever you write, modify, or refactor code, you MUST follow these documentation rules:
1. **Always add Docstrings**: Every module, class, method, and function must have a detailed docstring.
2. **Format**: Use the standard docstring format for the specific language (e.g., Google Style for Python).
3. **Content Requirements**:
   - Provide a clear, concise description of what the code does.
   - List all parameters (Args) with their expected types and purpose.
   - Describe the return value (Returns) and its type.
   - Note any exceptions or errors raised (Raises/Throws).
   - For complex functions, include a short usage example.

## Testing Guidelines
Write tests with `pytest`. Name files `test_*.py` and test functions
`test_*`. Mirror the source layout so coverage is easy to locate. Favor small,
deterministic unit tests with mocks for hardware, ZMQ, or timing-sensitive
code. Add regression tests for bug fixes, especially around interpolation,
shared memory, and robot interfaces.

## Commit & Pull Request Guidelines
This repository currently has no commit history, so no project-specific commit
convention is established yet. Use short, imperative commit subjects such as
`Add mock camera latency checks`. Keep commits focused. Pull requests should
describe the behavior change, list validation steps run locally, and include
logs or screenshots when a change affects camera output, recording, or robot
control flows.

---
> Source: [Richard-coder-Nai/HuMI](https://github.com/Richard-coder-Nai/HuMI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
