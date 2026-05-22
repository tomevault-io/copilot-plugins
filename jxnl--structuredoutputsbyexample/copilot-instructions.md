## structuredoutputsbyexample

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands
- Build static site: `python build_static_site.py`

## Code Style Guidelines
- Python version: >=3.9
- Formatting: Follow PEP 8, 4-space indentation, 88-100 character line length (Black compatible)
- Linting: Use ruff (`pip install ruff`)
- Type checking: Use mypy (`pip install mypy`)
- Import order: Standard library → Third-party → Local, alphabetically sorted within groups
- Naming: snake_case for functions/variables, PascalCase for classes
- Error handling: Use specific exception types with appropriate logging
- Comments: Use descriptive annotations for example code (these become documentation)

## Development Patterns
- Example files follow strict naming pattern: `NNN-example-name/example-name.py`
- Each example needs a .py file, .sh file, and _links.txt file
- Annotations in Python examples become documentation in the generated site
- Use type hints consistently throughout the codebase

## Example File Format
- Python example files MUST follow a specific comment format:
  - First line: Title comment (e.g., `# Getting Started with Structured Outputs`)
  - Immediately followed by description comment(s) with NO blank lines between comments
  - Example: 
    ```python
    # Title of Example
    # Description of the example that explains what it demonstrates.
    # Can span multiple lines if needed.
    ```
- The build script extracts the first comment line as the title and subsequent consecutive comment lines as the description
- Blank lines between comments disrupt the parsing and should NEVER be added
- Comments inside functions should use inline style for better documentation parsing

---
> Source: [jxnl/structuredoutputsbyexample](https://github.com/jxnl/structuredoutputsbyexample) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
