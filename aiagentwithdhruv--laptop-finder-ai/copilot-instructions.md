## 10-backend-fastapi

> FastAPI backend rules


Backend stack:
- FastAPI
- Pydantic
- SQLAlchemy or SQLModel
- PostgreSQL
- Redis for cache/queue support

Backend rules:
- Use clean architecture.
- Routes/controllers should only handle HTTP concerns.
- Business logic must live in services.
- Database access must live in repositories.
- Validation must be done using schemas/models.
- Use dependency injection patterns where appropriate.
- Use async I/O where supported and beneficial.
- Use pagination for list endpoints.
- Use consistent response models.
- Use centralized exception handling.

API conventions:
- Follow RESTful naming.
- Version APIs when needed, e.g. /api/v1/.
- Use proper status codes.
- Return predictable response structure.
- Do not leak internal stack traces or raw database errors.

Preferred backend structure:
- backend/api/routes/
- backend/api/dependencies/
- backend/services/
- backend/repositories/
- backend/models/
- backend/schemas/
- backend/core/
- backend/utils/
- backend/tests/

Coding expectations:
- Add type hints to functions.
- Add docstrings for non-trivial service methods.
- Keep routes thin and declarative.
- Keep service methods focused and testable.
- Reuse existing utilities before creating new ones.

Do not:
- Put SQL or ORM-heavy logic inside route files.
- Put business logic inside Pydantic schemas.
- Put environment variables directly across many files; centralize in config.
- Mix unrelated domains in the same service file.

---
> Source: [aiagentwithdhruv/laptop-finder-ai](https://github.com/aiagentwithdhruv/laptop-finder-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
