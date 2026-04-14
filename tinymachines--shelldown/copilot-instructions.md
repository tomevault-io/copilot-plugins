## shelldown

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build/Test Commands
- Install dev environment: `pip install -e . && pip install -r requirements.txt`
- Run: `python -m shelldown.cli <file.md>`
- Lint: `black shelldown`
- Format imports: `isort shelldown`
- Test all: `pytest`
- Test single: `pytest shelldown/tests/test_parser.py::test_parse_markdown_headings`
- Build package: `python setup.py sdist bdist_wheel`

## Code Style Guidelines
- Formatting: Use Black with 88 character line length
- Imports: Use isort with Black profile
- Types: Type hints where useful, especially in public API
- Naming: snake_case for Python variables/functions
- Error handling: Use descriptive error messages and proper error types
- Documentation: Docstrings for all functions, Google style format
- Tests: pytest, one test per function/behavior

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tinymachines)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tinymachines)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
