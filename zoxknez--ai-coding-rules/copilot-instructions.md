## 62-stack-python

> USE WHEN: working on Python/FastAPI services.


# Python / FastAPI

## Typing & Validation
- Use Pydantic v2 models for request/response schemas.
- Avoid raw dicts for external inputs.

## Async
- Use `async def` for I/O‑bound operations.
- Prefer `asyncpg` (or async‑capable drivers) for DB access.

## Style
- Prefer functions over classes unless stateful behavior is required.

---
> Source: [zoxknez/ai-coding-rules](https://github.com/zoxknez/ai-coding-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
