## python-rules

> - Avoid unnecessary complexity or deeply nested structures.


## Code Quality
- Avoid unnecessary complexity or deeply nested structures.
- Prefer built-in functions and idioms (e.g., list comprehensions, `with` for file handling).
- Use type hints (`def func(arg: str) -> int:`) and `mypy` for static analysis where appropriate.

## Error Handling
- Use `try/except` blocks to handle exceptions gracefully.
- Avoid catching broad exceptions like `except:` or `except Exception:` unless justified.

## Documentation
- Add docstrings to all public functions and classes using [PEP 257](https://peps.python.org/pep-0257/) conventions.
- Document parameters, return types, and side effects clearly.

## Dependency Management
- Use a `requirements.txt` or `pyproject.toml` for managing dependencies.
- Avoid unnecessary libraries when the standard library suffices.

## Testing
- Use `unittest`, `pytest`, or `doctest` for writing tests.
- Keep tests isolated, clear, and repeatable.

## Logging
- Use the built-in `logging` module instead of `print` for production code.
- Include log levels (`info`, `debug`, `warning`, `error`, `critical`) appropriately.

---
> Source: [consilience-dev/consensia-news-agent](https://github.com/consilience-dev/consensia-news-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
