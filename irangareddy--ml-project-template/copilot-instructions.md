## ml-project-template

> This document provides a summary of the project's structure, conventions, and common commands for the Gemini agent.

# Gemini Project Guide: ml-project-template

This document provides a summary of the project's structure, conventions, and common commands for the Gemini agent.

## 1. Project Overview

This is a machine learning (ML) / deep learning (DL) project configured with modern Python tooling. It uses `uv` for package management, `Ruff` for unified linting and formatting, and `pre-commit` for automated checks. All configuration is centralized in `pyproject.toml`.

## 2. Key Files

- **`pyproject.toml`**: Defines all project metadata, dependencies (production, optional, dev), and tool configurations (like Ruff).
- **`uv.lock`**: Pins exact versions of all dependencies for reproducibility. Commit this file to version control.
- **`.pre-commit-config.yaml`**: Configures the pre-commit hooks.
- **`Taskfile.yml`**: Defines common project commands.
- **`src/`**: Contains all the primary Python source code.
- **`tests/`**: Contains all the tests.
- **`notebooks/`**: Contains Jupyter notebooks.
- **`data/`**: Contains data files.

## 3. Common Commands (via Task)

This project uses [Task](https://taskfile.dev) to simplify common commands.

- **`task sync`**: Sync the environment with `uv.lock`.
- **`task install-hooks`**: Install the pre-commit hooks.
- **`task lint`**: Check for linting errors.
- **`task format`**: Format the codebase.
- **`task fix`**: Fix linting errors automatically.
- **`task test`**: Run tests.
- **`task run`**: Run the main application script.
- **`task serve`**: Serve the FastAPI application.
- **`task build`**: Build the project for distribution.

To see a list of all available tasks, run `task --list`.

## 4. Manual Commands

While `task` is recommended, you can also run the commands manually:

- **Sync environment from `uv.lock`**:

  ```bash
  uv sync
  ```

- **Run all tests**:

  ```bash
  uv run -m pytest
  ```

- **Run pre-commit checks**:

  ```bash
  uv run -- pre-commit run --all-files
  ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/irangareddy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/irangareddy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
