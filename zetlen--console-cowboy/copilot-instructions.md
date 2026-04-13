## console-cowboy

> **Hop terminals like you hop Linux distributions.**

# Console Cowboy

**Hop terminals like you hop Linux distributions.**

Console Cowboy is a CLI tool designed to make terminal configurations portable across different terminal emulators. It allows users to export settings from one terminal and import them into another using a Common Terminal Emulator Configuration (CTEC) intermediate format.

## Project Overview

*   **Type:** Python CLI Application
*   **Package Manager:** `uv` (primary), `pip`
*   **Supported Terminals:** iTerm2, Ghostty, Alacritty, Kitty, WezTerm, Terminal.app, VS Code.
*   **Core Logic:**
    *   **Adapters:** Uses a plugin-based adapter pattern (`TerminalAdapter`) to handle specific terminal formats.
    *   **CTEC:** An intermediate data model (dataclasses) that standardizes configuration (Colors, Fonts, Cursor, Window, etc.).
    *   **Serialization:** Supports YAML, JSON, and TOML.

## Building and Running

### Prerequisites
*   Python >= 3.10
*   `uv` (recommended)

### Installation
To install the project with development dependencies:

```bash
uv sync --all-extras
# OR
pip install -e '.[test,dev]'
```

### Running the CLI
You can run the CLI directly through `uv` or python module execution:

```bash
# Using uv
uv run console-cowboy --help

# Using python
python -m console_cowboy --help
```

### Testing
The project uses `pytest` for testing.

```bash
# Run all tests
uv run pytest

# Run with coverage
uv run pytest --cov=console_cowboy

# Run specific test file
uv run pytest tests/test_terminals.py
```

## Development Conventions

*   **Code Style:** The project uses `ruff` for linting and formatting.
    *   Line length: 88 characters.
    *   Quote style: Double quotes.
*   **Architecture:**
    *   **New Terminals:** To add a new terminal, create an adapter in `console_cowboy/terminals/` inheriting from `TerminalAdapter` and implement `parse()` and `export()`. Register it in `console_cowboy/terminals/__init__.py`.
    *   **Color Handling:** All colors should be normalized to RGB (0-255) using `utils/colors.py`.
    *   **Error Handling:** Use `ctec.add_warning()` for non-critical conversion issues instead of raising exceptions.
*   **Commit Messages:** Follow Conventional Commits (enforced by `commitizen`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zetlen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zetlen)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
