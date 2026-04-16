## my-python-template

> Your primary objective is to assist with development while strictly adhering to the established conventions and workflows of this project.

# Gemini Agent Instructions for my-python-template

Your primary objective is to assist with development while strictly adhering to the established conventions and workflows of this project.

## Core Principles

1.  **Trust the Project's Documentation**: Before taking any action, you MUST consult `docs/DEVELOPMENT_GUIDE.md` and `Taskfile.yaml`. These are your primary sources of truth. Do not rely on generic Python knowledge.
2.  **Use the Defined Toolchain**: This project uses a specific set of tools (`mise`, `uv`, `task`). You must use them as intended.

## Information Sources Hierarchy

When performing tasks, consult files in this order:
1.  **`GEMINI.md` (this file)**: For your own operational rules.
2.  **`Taskfile.yaml`**: To find the correct commands for routine tasks.
3.  **`docs/DEVELOPMENT_GUIDE.md`**: For detailed development workflows and conventions.
4.  **`README.md`**: For the project's overall purpose and quick start.

## Workflow Mandates & Rules

-   **Environment & Dependencies**:
    -   **DO NOT** use `pip`, `venv`, or `virtualenv` directly.
    -   **ALWAYS** use `uv` for Python environment and package management (e.g., `uv venv`, `uv sync`).
    -   To add a Python dependency, you **MUST** use `uv add <package>` or `uv add --dev <package>`.

-   **Code Quality & Testing**:
    -   **DO NOT** run `ruff`, `pytest`, `yamlfmt`, etc., directly.
    -   **ALWAYS** use `task` commands for all checks, formatting, and testing.
    -   To fix linting/formatting issues, **ALWAYS** attempt to use `task fix` first.
    -   After **ANY** code modification, you **MUST** run `task check` and `task test` to verify your changes.

-   **Tooling**:
    -   **DO NOT** attempt to install development tools manually.
    -   If a tool is missing, you should suggest editing `.mise.toml` and then running `mise install`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hyt-sasaki) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
