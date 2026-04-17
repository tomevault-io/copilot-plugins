## wled

> - Code Style: Follow PEP 8 guidelines (https://www.python.org/dev/peps/pep-0008/)

# Cursor Rules for Python Project

# General Project Settings
- Language: Python
- Code Style: Follow PEP 8 guidelines (https://www.python.org/dev/peps/pep-0008/)
- Maximum line length: 79 characters
- Indentation: 4 spaces (no tabs)
- Encoding: UTF-8
- Line endings: Unix-style (LF)

# File and Directory Structure
- Place Python source files (.py) in the `src/` directory
- Place unit tests in the `tests/` directory with filenames prefixed with `test_`
- Place configuration files (e.g., .ini, .yaml) in a `config/` directory
- Place documentation in the `docs/` directory
- Include a `README.md` in the project root
- Include a `.gitignore` file tailored for Python projects (e.g., ignore __pycache__, *.pyc, .venv/)

# Code Style and Conventions
- Use single quotes ('') for string literals unless double quotes ("") are necessary (e.g., for strings containing single quotes)
- Use descriptive variable and function names following snake_case convention
- Class names should use CamelCase
- Constants should be in ALL_CAPS with underscores
- Include docstrings for all modules, classes, and functions using triple quotes (""")
- Follow the order in module: imports, constants, classes, functions
- Group imports in this order: standard library, third-party, local project imports, separated by blank lines
- Use explicit relative imports (e.g., from .module import function) for local imports
- Avoid wildcard imports (e.g., from module import *)
- Include type hints for function parameters and return types (PEP 484)

# Error Handling and Logging
- Use specific exception handling (e.g., except ValueError instead of except Exception)
- Include meaningful error messages in exceptions
- Use the `logging` module instead of print() for debugging and logging
- Avoid bare except clauses

# Testing
- Use pytest for unit testing
- Write test functions with descriptive names starting with `test_`
- Place test files in the `tests/` directory
- Ensure test files are independent and include setup/teardown where necessary
- Aim for at least 80% test coverage

# Dependencies
- List all dependencies in a `requirements.txt` file
- Use virtual environments (e.g., venv or virtualenv) for dependency management
- Prefer pinned versions for dependencies (e.g., package==1.2.3)

# Documentation
- Use reStructuredText or Markdown for documentation
- Include docstrings for all public modules, classes, and functions
- Document function parameters, return values, and exceptions raised
- Maintain a `CHANGELOG.md` for tracking project changes

# Performance and Best Practices
- Avoid global variables unless absolutely necessary
- Use list comprehensions and generator expressions for concise and efficient code
- Prefer context managers (with statements) for resource management (e.g., file operations)
- Optimize imports to avoid loading unnecessary modules
- Use `__slots__` in classes for memory optimization when appropriate

# Security
- Avoid hardcoding sensitive information (e.g., API keys, passwords)
- Use environment variables for configuration (e.g., via `os.environ` or `python-dotenv`)
- Sanitize inputs to prevent injection attacks
- Use secure libraries for cryptographic operations (e.g., `hashlib`, `secrets`)

# Cursor-Specific Instructions
- Suggest code completions that adhere to PEP 8
- Warn about deprecated Python features or libraries
- Recommend type hints when generating or editing functions
- Suggest appropriate imports based on the project structure
- Highlight potential security issues in code suggestions

- Format code automatically on save using tools like Black or autopep8

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kamentr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
