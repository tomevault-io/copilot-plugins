## keel

> Python coding conventions and best practices


# Python Standards

Standards for Python development.

## Tooling

- Format with Ruff (or Black) — run `ruff format` to enforce consistent style
- Lint with Ruff (or Flake8): `ruff check --fix`
- Sort imports with Ruff's isort rules (or standalone `isort`)
- Type-check with `mypy --strict` (or `pyright`) — aim for zero type errors on new code
- Run `pytest` for tests

## Style

- Follow PEP 8 for formatting — use a formatter (Black or Ruff) to enforce it
- Use type hints for all function signatures and public APIs
- Maximum line length: 88 characters (Black default)
- Use `snake_case` for functions, methods, and variables
- Use `PascalCase` for classes
- Use `UPPER_SNAKE_CASE` for module-level constants
- Prefix private attributes with a single underscore (`_internal`)

## Imports

- Group imports in order: standard library → third-party → local
- Use absolute imports over relative imports
- Import modules, not individual names (prefer `import os` over `from os import path`)
- Use `isort` or Ruff to sort imports automatically
- Never use wildcard imports (`from module import *`)

## Functions

- Write small, focused functions with clear return types
- Use keyword arguments for functions with more than 3 parameters
- Use `*args` and `**kwargs` sparingly — prefer explicit parameters
- Return early to avoid deep nesting
- Use `typing.Optional` (or `X | None` in 3.10+) when `None` is a valid return

```python
def get_user(user_id: str, *, include_inactive: bool = False) -> User | None:
    """Retrieve a user by ID."""
    if not user_id:
        return None

    user = db.query(User).filter_by(id=user_id).first()
    if user and not user.active and not include_inactive:
        return None

    return user
```

## Error Handling

- Catch specific exceptions — never bare `except:` or `except Exception:`
- Use custom exception classes for domain-specific errors
- Use context managers (`with`) for resource management (files, connections, locks)
- Let unexpected exceptions propagate — don't catch and re-raise without adding context

## Data Structures

- Use dataclasses or Pydantic models for structured data — avoid raw dicts for domain objects
- Use `Enum` for fixed sets of values
- Prefer `collections.defaultdict`, `Counter`, and `namedtuple` over manual implementations
- Use list comprehensions for simple transformations; use loops for complex logic

## Testing

- Use `pytest` as the test framework
- Use fixtures for test setup and teardown
- Use `parametrize` for testing multiple inputs against the same logic
- Mock external services at the boundary — never mock internal implementation
- Name tests: `test_<function>_<scenario>_<expected_result>`

## Packaging

- Use `pyproject.toml` for project metadata and build configuration
- Pin direct dependencies; use ranges for library dependencies
- Include a `py.typed` marker for typed packages
- Use virtual environments — never install packages globally

## Agent Behavior

- After modifying `.py` files, run the project's formatter, linter, and type checker to verify correctness
- Fix all lint and type errors before presenting changes
- When creating new modules, include type hints on all public function signatures

## .gitignore

Ensure these Python-specific patterns are in the project's `.gitignore`:

```gitignore
__pycache__/
*.pyc
.venv/
venv/
.tox/
*.egg-info/
dist/
build/
.mypy_cache/
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/paulczar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
