## 03-backend-development

> FastAPI conventions, hardware integration, and asynchronous patterns.

# 03-backend-development

## Backend Development (FastAPI & Python)

STRICT REQUIREMENT:
- FastAPI Best Practices: Use decorators (`@app.get`, `@app.post`, `@app.websocket`), Pydantic models for request validation, and dependency injection for services.
- Hardware Integration: Abstract interactions via a common interface (`BaseSensor`) for extensibility and mocking.
- Asynchronous Operations: Utilize `asyncio` for non-blocking I/O.
- Security: Basic security measures defined in `app/core/security.py`.

---
> Source: [fivelity/usmp](https://github.com/fivelity/usmp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
