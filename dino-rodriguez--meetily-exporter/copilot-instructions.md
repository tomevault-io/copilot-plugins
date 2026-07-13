## meetily-exporter

> Meetily Exporter is a standalone Python CLI that reads Meetily's SQLite database and exports meetings as markdown files.

# CLAUDE.md

## Project

Meetily Exporter is a standalone Python CLI that reads Meetily's SQLite database and exports meetings as markdown files.

## Commands

Use the Makefile for all common tasks:

- `make test` — run tests
- `make lint` — run linter
- `make format` — auto-format code

## Python Style

### Typing
- Use full type hints on all public functions and methods.
- Prefer modern typing syntax: `list[str]`, `dict[str, int]`, `X | None`.

### Docstrings
- Use Google-style docstrings.
- Do not repeat types in docstrings — types belong in the signature.
- Docstrings explain behavior, intent, side effects, and errors.

Template:

```python
"""Short summary.

Optional details about behavior and edge cases.

Args:
    param: Description of purpose.

Returns:
    Description of what the caller receives.

Raises:
    ErrorType: When and why it happens.
"""
```

---
> Source: [dino-rodriguez/meetily-exporter](https://github.com/dino-rodriguez/meetily-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
