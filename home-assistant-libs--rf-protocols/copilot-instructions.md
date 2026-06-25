## rf-protocols

> Guidelines for agentic coding agents working in this repository.

# AGENTS.md — rf-protocols

Guidelines for agentic coding agents working in this repository.

---

## Project Overview

`rf-protocols` is a pure-Python library (Python >= 3.13) that encodes radio frequency
remote-control protocols into raw pulse/space timing sequences for OOK modulation. The
public API surface is small and intentionally minimal.

---

## Environment Setup

```bash
# Bootstrap dev environment (uses uv)
./script/setup.sh

# Equivalent manual step
uv pip install -e ".[dev]"
```

Dev extras install: `pytest`, `prek` (lint/format/type-check runner).

---

## Build / Lint / Test Commands

### Run all lint, format, and type-check hooks (changed files only)
```bash
prek
```

### Run all hooks on every file
```bash
prek --all-files
```

### Run linter only (with auto-fix)
```bash
ruff check --fix rf_protocols/
```

### Run formatter only
```bash
ruff format rf_protocols/
```

### Run type checker only
```bash
basedpyright rf_protocols/
```

### Run all tests
```bash
pytest --log-cli-level=debug
```

### Run a single test file
```bash
pytest tests/test_commands.py
```

### Run a single test by name
```bash
pytest tests/test_commands.py::test_command_defaults
```

> CI tests against Python 3.13 and 3.14 via GitHub Actions.

---

## Repository Structure

```
rf_protocols/            # Library source (only this directory is linted/type-checked)
    __init__.py          # Public API: defines __all__ and re-exports
    commands.py          # All domain logic: RadioFrequencyCommand ABC, ModulationType, Timing
tests/
    test_commands.py     # pytest suite (all tests in one file)
script/
    setup.sh             # Dev environment bootstrap
.pre-commit-config.yaml  # ruff, ruff-format, basedpyright hooks
pyproject.toml           # Build config, ruff rules, pyright settings
```

---

## Code Style

### Formatting
- Enforced by `ruff-format` (Black-compatible).
- **Indentation:** 4 spaces.
- **Quotes:** Double quotes (`"..."`).
- **Line length:** 88 characters (Black default).
- **Trailing commas:** Required in multi-line collections and argument lists.
- **Semicolons:** Never.
- Two blank lines between top-level definitions; one blank line between methods.

### Imports
- **Within the package:** use relative imports (`from .commands import ...`).
- **In tests:** use absolute imports from the installed package (`from rf_protocols import ...`).
- Named imports only; no wildcard imports (`from x import *`).
- Import order is enforced by ruff's `I` (isort) rules: stdlib -> third-party -> local.
- `__all__` must be defined in `__init__.py` to explicitly declare the public API.

### Naming
| Element | Convention | Example |
|---|---|---|
| Files / modules | `snake_case` | `commands.py` |
| Packages | `snake_case` | `rf_protocols` |
| Classes | `PascalCase` | `RadioFrequencyCommand`, `Timing` |
| Enums | `PascalCase` class, `UPPER_CASE` members | `ModulationType.OOK` |
| Functions / methods | `snake_case` | `get_raw_timings` |
| Variables / attributes | `snake_case` | `high_us`, `repeat_count` |
| Local numeric constants | `snake_case` (not `UPPER_CASE`) | `preamble_high = 2650` |
| Test functions | `test_<subject>_<description>` | `test_command_defaults` |
| Hex literals | Uppercase hex digits | `0xFF`, `0x04FB` |

### Types
- Type checker: `basedpyright` with `typeCheckingMode = "standard"`.
- **All** function parameters and return types must be annotated.
- `-> None` must be explicit on `__init__` and void methods.
- Use PEP 585 lowercase generics: `list[Timing]`, not `List[Timing]`.
- Use PEP 604 union syntax: `T | None`, not `Optional[T]`.
- Use `@override` (from `typing`, Python 3.12+) on every overridden method.
- No `Any`; avoid `cast`; prefer real type narrowing.
- Inline variable annotations where needed: `timings: list[Timing] = []`.

### Classes
- Abstract base classes use `abc.ABC` and `@abc.abstractmethod`.
- Immutable value objects use `@dataclass(frozen=True, slots=True)`.
- Constructor arguments should be **keyword-only** (use `*` separator) to prevent
  positional-argument confusion.

### Docstrings
- All public classes and methods must have a docstring.
- First line: concise one-line summary.
- Multi-line: blank line after the summary, then prose description. No Google/NumPy
  parameter sections unless complexity demands it.

---

## Architecture

### Adding a New Protocol
1. Subclass `RadioFrequencyCommand` (ABC) in `rf_protocols/commands.py` or a new module.
2. Implement `get_raw_timings(self) -> list[Timing]`.
3. Decorate the override with `@override`.
4. Define timing constants as local `snake_case` variables inside the method.
5. Re-export the new class from `rf_protocols/__init__.py` and add it to `__all__`.

### Key Abstractions
- **`Timing(high_us, low_us)`** -- frozen dataclass representing one pulse+space pair
  (microseconds). Immutable, comparable by value.
- **`ModulationType`** -- `StrEnum` of supported modulation types. Currently only `OOK`.
- **`RadioFrequencyCommand` (ABC)** -- base class for all RF protocol encoders. Holds
  `frequency`, `repeat_count`, `modulation`, `symbol_rate`, and `output_power`.

---

## Testing

- No mocking. Tests use pure value comparison against manually constructed
  `list[Timing]` fixtures.
- One assertion per logical case; reuse expected values with list unpacking
  (`[*expected[:-1], ...]`) rather than duplicating fixtures.
- Tests live in `tests/test_commands.py`; keep them in one file unless the suite
  grows substantially.
- Pre-commit hooks (`prek`) do **not** run on `tests/`; the type checker and linter
  only target `rf_protocols/`. Tests are still expected to be clean Python.

---

## Error Handling

- The library currently has no custom exceptions. Incorrect inputs surface as natural
  Python runtime errors (`TypeError`, `ValueError`).
- Correctness is enforced primarily through the type checker and immutable value
  objects rather than defensive runtime checks.

---
> Source: [home-assistant-libs/rf-protocols](https://github.com/home-assistant-libs/rf-protocols) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
