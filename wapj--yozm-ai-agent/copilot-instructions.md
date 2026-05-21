## yozm-ai-agent

> - Run Python script: `python filename.py`

# Claude Guidelines for yozm-ai-agent

## Commands
- Run Python script: `python filename.py`
- Run Jupyter notebook: `jupyter notebook` or `jupyter lab`
- Install dependencies: `pip install -e .` or `pip install -e ".[dev]"`
- Type checking: N/A (consider adding mypy)
- Linting: N/A (consider adding ruff or flake8)
- Tests: N/A (consider adding pytest)

## Code Style
- **Imports**: Group standard library, third-party, and local imports
- **Formatting**: Use 4 spaces for indentation
- **Naming**: snake_case for variables/functions, PascalCase for classes
- **String literals**: Use single quotes for simple strings
- **Error handling**: Use try/except for expected exceptions
- **Environment variables**: Load with python-dotenv, access via os.environ.get()
- **Logging**: Use Python's logging module with appropriate log levels
- **Comments**: Provide docstrings for functions/classes, include Korean comments for clarity
- **Main guard**: Always use `if __name__ == "__main__":`
- **Types**: Consider adding type hints for function parameters and return values

---
> Source: [wapj/yozm-ai-agent](https://github.com/wapj/yozm-ai-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
