## fastapi-react-backend-rules

> Rules for working with the React project in the /backend folder

This project should use the modern, concise, and best practices for building Python 3.11+ applications

# Primary Libraries

See [pyproject.toml](mdc:fastapi_react_app/pyproject.toml) for the specific libraries and versions you should use!
When making recommendations, make sure you are referencing the correct verions! Double check your work.

- FastAPI
- Pydantic 2.0+
- sqllite3

# Rules

- Use best practices for FastAPI.
- Use lowercase with underscores for directories and files (e.g., routers/user_routes.py).
- Prefer async functions for all I/O operations.
- Use type hints for all function signatures.
- Use Pydantic v2 models for input validation and response schemas.
- Implement sqlite3 and properly escaped SQL for CRUD operations
- Use Ruff for linting and formatting Python code.

---
> Source: [ai-evals-course/isaac-fasthtml-workshop](https://github.com/ai-evals-course/isaac-fasthtml-workshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
