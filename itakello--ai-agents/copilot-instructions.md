## ai-agents

> Python Coding Guidelines


## 1. Style & Formatting
- **PEP 8**: Strictly adhere to PEP 8 style guidelines. Use linters like Ruff and formatters like Black to enforce this.
- **Naming**: Follow PEP 8 naming conventions (e.g., `snake_case` for functions and variables, `CapWords` for classes).
- **Imports**:
    - Organize imports per PEP 8: standard library, third-party, local application/library.
    - Use absolute imports by default.
    - Avoid `from module import *`.

## 2. Type Hinting (Python 3.10+)
- **Mandatory Typing**: Use type hints for all function signatures (arguments and return types) and variable declarations where appropriate.
- **Modern Syntax**:
    - Prefer `list` over `typing.List`, `dict` over `typing.Dict` and `set` over `typing.Set`.
    - Use `|` for union types (e.g., `int | str`) instead of `typing.Union`.
    - Use `str | None` for optional types instead of `typing.Optional[str]`.
- **Pydantic**:
    - Use `pydantic.BaseModel` for data validation, serialization, and settings management, especially for external data.
    - Document `pydantic` model fields with `Field(description="...")`.
- **Dataclasses**:
    - Use `@dataclasses.dataclass(kw_only=True)` for simple, internal data containers not requiring `pydantic`'s advanced validation.

## 3. Best Practices
- **`pathlib`**: Use the `pathlib` module for all file system path manipulations.
- **List Comprehensions/Generator Expressions**: Prefer these over `map()` and `filter()` for readability when appropriate.
- **Exception Handling**: Be specific in exception handling (e.g., `except ValueError:` instead of `except Exception:`).
- **Logging**: Utilize the standard `logging` module or a structured logging library (e.g., `loguru` if adopted project-wide). Configure loggers effectively.
- **Progress Bars**: For long-running loops, consider `tqdm` for user feedback.
- **Class Structure**: Place public methods/attributes before private ones (prefixed with `_`).

## 4. Project Structure & Entry Points
- **Clear Structure**: Organize projects logically (e.g., `src/` layout).
- **Entry Points**: For applications, consider a `src/main.py` or use `pyproject.toml` defined scripts.
- **Modularity**: Design modules with clear responsibilities.

## 5. Testing (with Pytest)
- **Test Location**: Keep tests in a dedicated `tests/` directory, mirroring the `src/` structure.
- **Naming**: Test files should be `test_*.py` or `*_test.py`. Test functions should be prefixed with `test_`.
- **Assertions**: Use descriptive `pytest` assertions.
- **Fixtures**: Leverage `pytest` fixtures for test setup and teardown.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Itakello) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
