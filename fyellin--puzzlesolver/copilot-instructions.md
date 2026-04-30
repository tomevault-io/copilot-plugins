## puzzlesolver

> Preliminary Python coding guidance


# Python Preliminary Rule

Use this as a starter rule and customize as needed. Match the style of `solver/` unless a file already follows a different local convention.

## Core expectations

- Prefer clear, readable code over clever shortcuts.
- Keep functions focused and small when practical.
- Add type hints for new or changed public functions.
- Handle errors explicitly; avoid silent failures.
- Automatically update the imports in the style expected by ruff. Delete any unused imports as necessary.
- Do NOT format using "ruff format". I do not like each argument of a function and each
  element of a list on a separate line. I only want the imports to be in ruff format.

## Style reference: `solver/` package

Summarize and follow this when editing Python in this repo:

- **Imports**: Standard library first, then a blank line, then project imports (`from .module import ...`). Keep imports sorted and grouped the way ruff expects; do not run full-file `ruff format` on argument lists or collections.
- **Typing**: Use modern annotations throughout (`list[str]`, `X | None`, `Sequence[T]` from `collections.abc`). Use `type Name = ...` for common dict/alias types. Prefer `collections.abc` (`Sequence`, `Mapping`, `Callable`, `Iterable`) over concrete `list`/`dict` in public signatures when appropriate. Use `Protocol` for structural types, `ABC` / `@abstractmethod` when inheritance is intentional. Generic classes use PEP 695 style (e.g. `class Clue[T]:`).
- **Protocols (explicit structural subtyping)**: When a concrete class is meant to satisfy a `Protocol`, **list that protocol as a base class** in the class header so the contract is explicit (e.g. `class MyModel(SomeProtocol, ABC):` or `class MyValue(SomeProtocol):`). Do not rely on duck typing alone for documented protocol implementations. Combine with `ABC` when you also use `@abstractmethod`. Mark protocols with `@typing.runtime_checkable` when you need `isinstance(..., ThatProtocol)` at runtime (required for `isinstance` checks against protocols in current Python).
- **Classes**: Declare instance attributes with class-body type annotations where useful. Use a single leading underscore for module- or class-private names intended as implementation details.
- **Naming**: `snake_case` for functions and variables; `PascalCase` for classes; module-level constants in `UPPER_SNAKE` when they are true constants.
- **Docstrings**: Short docstrings on non-trivial public methods and on modules or types where behavior is not obvious from the name alone.
- **Comments**: Use comments for invariants, non-obvious algorithm choices, or solver-domain quirks; avoid restating what the code already says.
- **Public API**: Package `__init__.py` uses `__all__` to list exports; mirror that pattern when adding re-exports.
- **Line breaks**: Prefer natural wrapping (continuation indent) over forcing every argument or list element onto its own line; keep density similar to existing `solver/` files.

## Project workflow

- Reuse existing project patterns before introducing new ones.
- Keep changes scoped to the request; avoid unrelated refactors.
- Add or update tests when behavior changes.

## Quality checks

- Run relevant tests for touched Python areas before finishing.
- If linting/formatting tools are configured, run them on changed files.

## Notes for future edits

- Tighten or extend the `solver/` reference above if conventions drift.
- Add framework/library conventions here when they differ from `solver/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fyellin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
