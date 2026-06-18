## pde-constrained-optimization

> Core Python scripts live in `src/`. Current modules include `heat_equation.py` for the forward heat-equation example, `population_system.py` for reproducing the ODE parameter-estimation example, and `generate_solutions.py` for solution generation utilities. Rendered plots and exported figures go in `output/`; keep generated assets there rather than alongside source files. Top-level reference material such as `System_ODE_Parameter_est.html` and project notes in `README.md` document the mathematical background.

# Repository Guidelines

## Project Structure & Module Organization
Core Python scripts live in `src/`. Current modules include `heat_equation.py` for the forward heat-equation example, `population_system.py` for reproducing the ODE parameter-estimation example, and `generate_solutions.py` for solution generation utilities. Rendered plots and exported figures go in `output/`; keep generated assets there rather than alongside source files. Top-level reference material such as `System_ODE_Parameter_est.html` and project notes in `README.md` document the mathematical background.

## Build, Test, and Development Commands
This project uses `uv` with Python 3.12+.

- `uv sync`: install project dependencies from `pyproject.toml` and `uv.lock`.
- `uv run python src/heat_equation.py`: run the heat-equation example and regenerate output figures.
- `uv run python src/population_system.py`: run the population-system reproduction script.
- `uv run python src/generate_solutions.py`: execute helper solution-generation code.

If you add a new experiment, prefer exposing it as a runnable script under `src/`.

## Coding Style & Naming Conventions
Use 4-space indentation and standard PEP 8 Python style. Prefer `snake_case` for functions, variables, and filenames; reserve `UPPER_CASE` for module-level constants such as output directories. Keep numerical code explicit and readable: short helper functions for model terms, clear array shapes, and brief comments only where the math is not obvious from the code.

## Testing Guidelines
There is no formal test suite yet. For now, validate changes by running the relevant script with `uv run python ...` and checking that figures, arrays, and parameter estimates are sensible. When adding tests, place them in a top-level `tests/` directory and use names like `test_heat_equation.py`. Focus on deterministic checks for solvers, residual assembly, and parameter-recovery routines.

## Commit & Pull Request Guidelines
Recent commits use short, descriptive messages in sentence case, for example: `Starting off with creating solutions to the heat equation so that the thermal diffusivity constant can be calculated later on`. Keep commit messages specific to the mathematical or implementation change. Pull requests should summarize the model or numerical change, list commands used for validation, and include updated plots when output figures change.

## Scientific Workflow Notes
Prefer synthetic-data experiments before moving to inverse problems with unknown parameters. Keep derivations aligned with the reference paper and document any intentional deviations in code comments or the PR description.

---
> Source: [dkrawciw/PDE-Constrained-Optimization](https://github.com/dkrawciw/PDE-Constrained-Optimization) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
