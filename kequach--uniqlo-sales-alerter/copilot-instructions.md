## python-standards

> Python coding standards for source files


# Python Standards

## Naming

- Avoid single-letter variables outside list comprehensions and lambdas.
- Use descriptive names: `safe_title` not `t`, `color_display_code` not `cd`.
- Abbreviations `pid` (product ID), `pg` (price group), `wv` (watched variant) are acceptable as they are domain-standard in this codebase.

## Constants

- Extract magic strings and numbers into module-level constants (e.g. `_STOCK_OUT`, `_DEFAULT_PRICE_GROUP`, `_SMTP_TIMEOUT`).
- Use `frozenset` for immutable sets of valid values.

## Documentation

- All public classes and functions must have docstrings.
- Private helpers (`_name`) should have docstrings if their purpose isn't obvious from the name.
- Do NOT add comments that narrate what the code does. Comments should explain *why*.

## Logging

- Use `logger` for diagnostic output, not `print()`.
- Exception: `print()` is acceptable for intentional user-facing console output (rate-limit warnings, CLI preview).

## Error handling

- Prefer specific exception types over bare `except Exception`.
- When a missing optional dependency is detected, be consistent: log and return (like Telegram) or raise with a clear message (like email). Don't mix approaches for the same severity.

## Pydantic models

- All model classes need a class docstring.
- Use `Field(default_factory=...)` for mutable defaults, never `= []` or `= {}`.

---
> Source: [kequach/uniqlo-sales-alerter](https://github.com/kequach/uniqlo-sales-alerter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
