## python

> - Use Python 3.10 minimum

# Python Development Rules

## Language Requirements
- Use Python 3.10 minimum
- Leverage Python 3.10+ features when appropriate:
  - `match-case` statements for complex conditionals
  - Union types with `|` syntax: `str | None` instead of `Optional[str]`
  - Structural pattern matching
  - Parameter specification variables (`ParamSpec`)

## Code Style & Structure

### Control Flow
- **Always prefer guard clauses and early returns** over nested conditionals
- Use `return`, `continue`, or `break` early to handle edge cases and error conditions
- Avoid deep nesting - flatten code structure with guard clauses
- Maximum nesting depth: 2-3 levels

**Preferred pattern:**
```python
def process_data(data: dict | None) -> str | None:
    if not data:
        return None
    
    if 'required_field' not in data:
        return None
    
    # Main logic here
    return process_result(data)
```

**Avoid this anti-pattern:**
```python
def process_data(data: dict | None) -> str | None:
    if data:
        if 'required_field' in data:
            # Main logic deeply nested
            return process_result(data)
    return None
```

### Function Design
- Keep functions focused with single responsibilities (max 20-30 lines)
- Use guard clauses to validate inputs early
- Prefer pure functions when possible (no side effects)
- Use descriptive names that clearly indicate purpose
- When dealing with Frontend data, prefer API-based models which return JSON instead of forms and request parsing.

### Type Annotations
- Use type hints consistently throughout codebase
- Prefer Python 3.10+ union syntax: `str | None` over `Optional[str]`
- Use `TypeAlias` for complex type definitions
- Consider using `Protocol` for structural typing
- Use `Final` for constants that shouldn't be reassigned

### Error Handling
- Prefer specific exceptions over generic `Exception`
- Use guard clauses for input validation
- Fail fast - validate early, return/raise immediately on invalid input
- Consider using `Result` pattern for functions that may fail

## Package Management (uv)
- **Prefer `uv` over pip** for all dependency management
- Use `uv add package-name` instead of `pip install`
- Use `uv add --dev package-name` for development dependencies
- Use `uv run script.py` for executing scripts in project environment
- Use `uv sync` for installing dependencies from lock files
- Create virtual environments with `uv venv .venv`
- Pin exact versions in production: `uv add "package==1.2.3"`

## Project Structure
```
project/
├── pyproject.toml          # Project configuration
├── uv.lock                 # Dependency lock file (commit this)
├── README.md               # Project documentation
├── src/
│   └── package_name/       # Source code
└── tests/                  # Test files
```

### Configuration Files
- **Always include `pyproject.toml`** for project configuration
- Specify Python version requirement: `requires-python = ">=3.10"`
- Use `uv.lock` for dependency locking (commit to version control)
- Configure linting tools (ruff, mypy) in pyproject.toml

### Example pyproject.toml:
```toml
[project]
name = "your-project"
version = "0.1.0"
requires-python = ">=3.10"
dependencies = []

[project.optional-dependencies]
dev = ["pytest", "ruff", "mypy"]

[tool.ruff]
line-length = 88
target-version = "py310"

[tool.mypy]
python_version = "3.10"
strict = true
```

## Code Organization Principles
- **Prefer composition over inheritance** - use dependency injection
- Keep classes small and focused (Single Responsibility Principle)
- Use dataclasses or Pydantic models for data structures
- Group related functionality into modules
- Use `__init__.py` to control public API exposure

## Performance & Best Practices
- Use f-strings for string formatting
- Prefer list/dict comprehensions over loops when readable
- Use `pathlib` instead of `os.path` for file operations
- Consider using `functools.lru_cache` for expensive pure functions
- Use context managers (`with` statements) for resource management

## Testing Guidelines
- Write tests alongside code development
- Use descriptive test names that explain the scenario
- Follow AAA pattern: Arrange, Act, Assert
- Use pytest fixtures for test setup
- Aim for high test coverage but focus on critical paths

## Documentation
- Use docstrings for all public functions/classes
- Follow Google or NumPy docstring style consistently
- Include type information in docstrings when helpful
- Keep README.md updated with setup and usage instructions

---
> Source: [mez-0/citadel](https://github.com/mez-0/citadel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
