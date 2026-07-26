## openagent

> Security, auth, and async tasks for LMForge API

Security & Async:
- Rate limiting: use existing Flask-Limiter patterns.
- Background tasks (>2s): ALWAYS use Celery with Redis broker (DB 1).
- Database: use SQLAlchemy sessions via Injector, always commit/rollback properly.
- Logging: use the project's structlog configuration.
- Validation: Pydantic for all inputs/outputs.

---
> Source: [Haohao-end/openagent](https://github.com/Haohao-end/openagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
