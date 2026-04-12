## mycosoft-mas

> Python coding standards for the MAS codebase


# Python Coding Standards (MAS)

## General

- Python 3.13, managed by Poetry
- Use async/await for all I/O operations
- Type hints required on all function signatures
- Use `from __future__ import annotations` when needed
- Format with `black`, sort imports with `isort` (`make fmt`)

## Error Handling

```python
# Always use specific exceptions, never bare except
try:
    result = await client.fetch(url)
except httpx.HTTPError as e:
    logger.error("HTTP request failed", url=url, error=str(e))
    raise
except Exception as e:
    logger.error("Unexpected error", error=str(e))
    raise
```

## Logging

- Use structured logging: `logger.info("message", key=value)`
- Never use `print()` in production code

## Data

- NEVER use mock/fake/placeholder data
- All data must come from real services (MINDEX, Redis, external APIs)
- If a service is unavailable, raise proper exceptions

## Naming

- Classes: `PascalCase`
- Functions/methods: `snake_case`
- Constants: `UPPER_SNAKE_CASE`
- Files: `snake_case.py`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MycosoftLabs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MycosoftLabs)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
