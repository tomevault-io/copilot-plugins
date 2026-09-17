## prp-compiler

> This guide provides instructions for developing and contributing to the `prp-compiler` project. It assumes you have already set up the project using the instructions in the `README.md`.

# Contributor Guide

This guide provides instructions for developing and contributing to the `prp-compiler` project. It assumes you have already set up the project using the instructions in the `README.md`.

## Dev Environment Tips
- Use `uv run <script_name>` to execute predefined scripts from `pyproject.toml`. Key scripts include:
  - `uv run pytest -v`: Run the full test suite.
  - `uv run lint`: Run the Ruff linter to check for style issues.
  - `uv run validate`: Run mypy for static type checking.
- Run `uv sync` any time you pull changes that might have updated the `pyproject.toml` to ensure your local environment has all the required dependencies.
- The project's name for configuration purposes is `prp-compiler`, as defined in the `[project].name` field of `pyproject.toml`.

## Testing Instructions
- Run the entire test suite using the following command. All tests must pass before merging a pull request.
  ```bash
  uv run pytest
  ```
- To run a specific test file or a subset of tests, you can pass arguments to pytest. For example, to run only tests containing "golden" in their name:
  ```bash
  uv run pytest -k "golden"
  ```
- To focus on a single test function, use the `-k` flag with the test name:
  ```bash
  uv run pytest -k "test_golden_prp"
  ```
- After making changes, especially to file structures or dependencies, run the linter and type checker to ensure code quality:
  ```bash
  uv run lint
  uv run validate
  ```
- Please add or update tests for any code you change. Contributions that improve test coverage are highly valued.

## PR Instructions
- **Title Format**: Please use the following format for your pull request titles to ensure clarity and consistency.
  ```
  [prp-compiler] <A brief, descriptive title of your changes>
  ```

---
> Source: [eddiedunn/prp_compiler](https://github.com/eddiedunn/prp_compiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
