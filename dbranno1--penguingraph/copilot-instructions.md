## penguingraph

> All Python operations in this repository must use [`uv`](https://docs.astral.sh/uv/) for package management, environment isolation, and script execution.

# Agent Guidelines

## Python Development with `uv`

All Python operations in this repository must use [`uv`](https://docs.astral.sh/uv/) for package management, environment isolation, and script execution.

### Rules

1. **Running Python Scripts**:
   - Always run scripts using `uv run`:
     ```bash
     uv run python <script_name>.py
     # or
     uv run <script_name>.py
     ```
   - Do not invoke bare `python` or `py` directly.

2. **Managing Packages & Dependencies**:
   - **Add dependencies**: `uv add <package>` (or `uv pip install <package>`)
   - **Add dev dependencies**: `uv add --dev <package>`
   - **Remove dependencies**: `uv remove <package>`
   - **Sync environment**: `uv sync`
   - Do not invoke `pip install` directly.

3. **Running Tools & Tests**:
   - Always execute test runners, linters, and formatters with `uv run`:
     ```bash
     uv run pytest
     uv run ruff check .
     uv run ruff format .
     ```
   - For ephemeral / standalone CLI tools, use `uvx <tool>`.

4. **Environment Setup**:
   - Create or initialize virtual environments via `uv`:
     ```bash
     uv venv
     ```

---
> Source: [dbranno1/PenguinGraph](https://github.com/dbranno1/PenguinGraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
