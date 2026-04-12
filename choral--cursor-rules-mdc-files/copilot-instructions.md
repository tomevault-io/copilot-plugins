## cursor-rules-mdc-files

> FastAPI design principles and patterns for building APIs


# FastAPI Design Principles
- Use proper dependency injection with FastAPI's `Depends`.
- Avoid technical debt by steering clear of unmaintained or deprecated packages.
- Implement middleware for logging, validation, and error handling to enhance code maintainability.
- Follow modular organization using the following architectural companion rules:
  - @fastapi-router.mdc
  - @fastapi-schemas.mdc
  - @fastapi-models.mdc
  - @fastapi-dal.mdc
  - @fastapi-alembic-migration.mdc

# Design Principles
- Separation of Concerns: Organize code into routers, services, and repositories.
- Single Responsibility Principle: Each component should have a focused responsibility.
- Don't Repeat Yourself: Abstract shared logic into reusable services or utilities.
- Keep It Simple: Prioritize clarity and simplicity over complexity.
- Open/Closed Principle: Code should be open for extension but closed for modification.

# Design Patterns
- Router Pattern: Use `APIRouter` to group and modularize routes by feature or domain.
- Dependency Injection: Use FastAPI’s `Depends` to inject services and database sessions.
- Repository Pattern: Encapsulate database logic in repository classes.
- Service Layer Pattern: Centralize business logic in service classes.
- Factory Pattern: Use factory functions to initialize shared resources (e.g., DB sessions).
- DTO Pattern: Use Pydantic models for input/output data validation.

# Project Structure
- `main.py`: FastAPI entry point; sets up the app and includes routers.
- `app/routers/`: Domain-specific route definitions.
- `app/services/`: Business logic implementations.
- `app/dal/`: Data access layer, separated from logic. More information can be found at companion rule [fastapi-dal.mdc](mdc:.cursor/rules/fastapi-dal.mdc)
- `app/schemas/`: Pydantic models for request and response. More information can be found at companion rule @fastapi-schemas.mdc
- `app/orm_models/`: SQLAlchemy ORM models. More information can be found at companion rule @fastapi-models.mdc
- `app/core/`: Shared utilities, config management, and dependency setup.
- `alembic`: Contains the migration code using alembic framework. More information can be found at companion rule [fastapi-alembic-migration.mdc](mdc:.cursor/rules/fastapi-alembic-migration.mdc)

# Implementation Notes
- Use `Depends()` to manage dependencies across routes and services.
- Use decorator `@router.post()` for route definitions.
- Use `BaseSettings` from Pydantic to manage application settings and secrets.
- Use factories for initializing services, sessions, or third-party clients.
- Maintain modularity to ensure testability and clarity.

# Extensibility
- Implement middleware for logging, security, and request tracing.
- Use `BackgroundTasks` for asynchronous background jobs.
- Adopt async functions for I/O-bound operations when supported.

# Testing Strategy
- Write unit tests for service and repository components.
- Use `TestClient` from FastAPI for integration and functional tests.
- Mock dependencies with `unittest.mock` or `pytest-mock`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/choral)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/choral)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
