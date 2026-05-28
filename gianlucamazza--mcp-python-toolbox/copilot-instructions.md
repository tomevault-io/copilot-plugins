## mcp-python-toolbox

> - **Install dev dependencies**: `pip install -e ".[dev]"`

# MCP Python Toolbox - Development Guidelines

## Build/Test/Lint Commands
- **Install dev dependencies**: `pip install -e ".[dev]"`
- **Run all tests**: `pytest`
- **Run single test**: `pytest tests/path/to/test_file.py::test_function_name -v`
- **Type checking**: `mypy src/mcp_python_toolbox`
- **Linting**: `pylint src/mcp_python_toolbox`
- **Format code**: `black src/mcp_python_toolbox`

## Code Style Guidelines
- **Line length**: 100 characters (Black configuration)
- **Python version**: 3.7+
- **Formatting**: Use Black for automatic formatting
- **Imports**: Standard library first, then third-party, then local modules
- **Types**: All functions must have type annotations (disallow_untyped_defs=true)
- **Naming**: 
  - Classes: PascalCase
  - Functions/variables: snake_case
  - Constants: UPPER_CASE
- **Error handling**: Use specific exceptions with descriptive error messages
- **Documentation**: Docstrings for all modules, classes, and functions using """triple quotes"""

---
> Source: [gianlucamazza/mcp_python_toolbox](https://github.com/gianlucamazza/mcp_python_toolbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
