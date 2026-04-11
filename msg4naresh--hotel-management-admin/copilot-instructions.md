## hotel-management-admin

> This project is a FastAPI-based backend API for managing hotel rooms, customers, bookings, and document uploads. It uses PostgreSQL as the database and AWS S3 for file storage. The project is well-structured, with a clear separation of concerns between API endpoints, business logic, data models, and services.

# Hotel Management System - Admin API

## Project Overview

This project is a FastAPI-based backend API for managing hotel rooms, customers, bookings, and document uploads. It uses PostgreSQL as the database and AWS S3 for file storage. The project is well-structured, with a clear separation of concerns between API endpoints, business logic, data models, and services.

## Building and Running

### Prerequisites

- Python 3.10+
- [UV](https://github.com/astral-sh/uv) (or pip)
- Docker

### Development

1.  **Install Dependencies:**
    ```bash
    uv sync
    ```

2.  **Run Database Migrations:**
    ```bash
    uv run alembic upgrade head
    ```

3.  **Start the Development Server:**
    ```bash
    uv run uvicorn app.main:app --reload
    ```
    The API will be available at `http://localhost:8000/docs`.

### Docker

To run the application using Docker, use the following command:

```bash
docker-compose up -d
```

The API will be available at `http://localhost:8050/docs`.

### Testing

Run the test suite with the following command:

```bash
uv run pytest tests/ -v
```

## Development Conventions

- **Code Style:** The project uses `black` for code formatting and `isort` for import sorting.
- **Linting:** `ruff` is used for linting.
- **Type Checking:** `mypy` is used for static type checking.
- **Database Migrations:** `alembic` is used for managing database schema migrations.
- **Dependencies:** `uv` is used for package management.

## Key Files

- **`app/main.py`:** The entry point of the FastAPI application.
- **`app/api/routes.py`:** The main API router.
- **`app/api/endpoints/`:** Contains the API endpoints for different resources (bookings, rooms, etc.).
- **`app/models/`:** Contains the SQLAlchemy database models and Pydantic schemas.
- **`app/services/`:** Contains the business logic for services like S3 interaction.
- **`app/db/`:** Contains the database connection and initialization logic.
- **`pyproject.toml`:** Defines the project dependencies and tool configurations.
- **`alembic/`:** Contains the database migration scripts.
- **`tests/`:** Contains the unit and integration tests.
- **`docker-compose.yml`:** Defines the Docker services for the application and database.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/msg4naresh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/msg4naresh)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
