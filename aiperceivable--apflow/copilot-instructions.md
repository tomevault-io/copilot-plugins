## code

> High-Quality Code Specification – Simplicity First


## Core Principles
- Simplicity, readability, maintainability highest priority.
- No premature abstraction, optimization, or over-engineering.
- Code must be clear in ≤10 seconds; prefer simple over "elegant".

## Python Code Quality

### Readability
- Precise full-word names (limited standard abbreviations).
- Functions ≤50 lines, single responsibility, verb-named.
- No obscure tricks or overuse of comprehensions/*args/**kwargs/decorators.
- Break complex logic into small functions.

### Types (Mandatory)
- Full type annotations.
- No `Any` except dynamic JSON/third-party.
- Use `dataclass`, `TypedDict`, `Protocol`, `NewType`.

### Design
- Functional + data classes preferred; avoid deep inheritance.
- Composition > inheritance; ABC/Protocol only for real multiples.
- No circular imports.
- Dependency injection for config/logging/DB.

### Errors & Resources
- Explicit exception handling; no bare `except:`.
- Context managers for resources.
- Validate all public inputs.

### Logging
- `info` for critical paths; `error` with context for exceptions.
- No `print()` debugging.

### Testing
- Unit tests for core logic in `tests/`; ≥90% coverage.
- Names: `test_<feature>_<behavior>`.
- No changes without tests.

### Performance & Security
- No unjustified O(n²)+ in hot paths.
- Sanitize user input.
- Secrets only from config/secret management.

### Cursor Rules
- Never auto-create docs or examples.
- No extra `.pyi` or bloated `__init__.py`.
- English only for comments/docstrings/logs/errors.
- Fully understand context before logic changes.

### Formatting
- Zero errors: `ruff` + `black` + `pyright`.
- Imports sorted: std → third-party → local.

---
> Source: [aiperceivable/apflow](https://github.com/aiperceivable/apflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
