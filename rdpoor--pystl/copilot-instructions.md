## pystl

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Parameterized 3D printing of parts using [SolidPython2](https://github.com/jeff-dh/SolidPython), a Python library that generates OpenSCAD models programmatically.

## Tooling

- **Linter/formatter:** Ruff
- **Type checker:** mypy
- **Tests:** pytest
- **Package manager:** uv

```bash
uv sync                                                  # install deps
uv run python scripts/boilerplate.py                     # render example (MyCustomPart cube)
uv run python scripts/boilerplate.py --help              # show available parameters
uv run pytest                                            # run all tests
uv run ruff check . && uv run ruff format .              # lint + format
uv run mypy src/                                         # type check
```

## Architecture Notes

The expected pattern for SolidPython2 projects:

- `scripts/<my_part>.py` — each custom part is a self-contained script; copy from `scripts/boilerplate.py`
- Parameters are `@dataclass` fields on a `PyStlPart` subclass; every field becomes a CLI flag automatically
- `build() -> OpenSCADObject` is the method subclasses implement; the inherited `render(output_dir)` writes `.scad` and (if `openscad` is in PATH) `.stl`
- `MyCustomPart.build()` typically composes reusable sub-parts from `src/pystl/library/`

### Adding a Library Sub-part

1. Create `src/pystl/library/<part_name>.py` with a `@dataclass` subclassing `PyStlPart` and implementing `build() -> OpenSCADObject`
2. Import it in `src/pystl/library/__init__.py`

### Adding a Custom Part Script

1. Copy `scripts/boilerplate.py` to `scripts/<my_part>.py`
2. Rename `MyCustomPart`, implement `build()` (composing library sub-parts as needed), and extend argparse in `main()` if required
3. Run `uv run python scripts/<my_part>.py` to render
4. Run `uv run python scripts/test_library_part.py --part <ClassName>` to run a quick render smoke-test

## Development guidelines for python-based projects

### Logging
- Always use Python's `logging` module. Never use bare `print()` statements in source files or scripts.
- Library code (`src/`): `log = logging.getLogger(__name__)` at module level.
- Scripts (`scripts/`): configure the root logger at startup via a `setup_logging()` function; silence noisy third-party loggers (`yfinance`, `urllib3`, `httpx`, etc.).
- Use appropriate levels: `DEBUG` for per-item detail, `INFO` for progress and summaries, `WARNING` for recoverable issues (rate limits, retries), `ERROR` for failures.

### Python Style Guidelines

Adhere strictly to the established Python style guidelines, which are based on [e.g., Google's Python Style Guide](https://google.github.io/styleguide/pyguide.html) or [PEP 8](https://peps.python.org) style guidelines.

*   **PEP 8:** Follow all PEP 8 conventions for maximum readability and consistency.
*   **Naming Conventions:** Use `snake_case` for function and variable names, `CapitalCase` for class names, and `UPPER_CASE` for constants.
*   **Type Annotations:** Must use Python's modern type annotations (PEP 484) consistently.
*   **Docstrings:** Use [Google-style docstrings](https://mcpmarket.com/tools/skills/python-style-guide-1) for all functions, classes, and modules.
*   **String Formatting:** Prefer `f-strings` for string formatting.
*   **Resource Management:** Use `with` statements (context managers) for file and resource handling.
*   **Comparisons:** Use `is` when comparing with `None`, `True`, or `False`.
*   **Comprehensions:** Use list, dictionary, and set comprehensions, and generator expressions for efficient data manipulation.
*   **Error Handling:** Employ robust error and exception handling, favoring the EAFP (Easier to Ask Forgiveness than Permission) principle.
*   **Imports:** Structure imports according to PEP 8 (standard library first, then third-party, then local imports), using absolute imports and grouping them with blank lines.

### Workflow & Verification
When proposing changes or generating new code, you must follow these steps to ensure quality:

1.  **Format Code:** After generation, run the configured format command (e.g., `uv run ruff format .`) to ensure style consistency.
2.  **Run Checks:** Execute the linting and type-checking scripts to verify correctness (e.g., `pylint`, `mypy`, `flake8`).
3.  **Run Tests:** Create or update unit tests to cover new functionality and edge cases.

### Project Structure
*   `src/`: Main source code directory.
*   `tests/`: Unit tests and integration tests.
*   `docs/`: Project documentation.
*   `pyproject.toml` or `requirements.txt`: Project dependencies and configuration files.

### Communication Style
*   Be concise and direct in your responses.
*   Lead with the answer or the proposed code change, then provide explanations.
*   If uncertain about something, state the uncertainty rather than hallucinating information.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rdpoor)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rdpoor)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
