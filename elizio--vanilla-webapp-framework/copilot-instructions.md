## backend-rules

> Flask API, SQLAlchemy, JWT, and pytest conventions for backend Python code

## **1. Backend Development Rules**
### **Flask API Development**
- **Rule 1.1**: Always use **Flask Blueprints** for modular API routing.
- **Rule 1.2**: Follow RESTful conventions (`GET`, `POST`, `PUT`, `DELETE`) for endpoints.
- **Rule 1.3**: Use **SQLAlchemy ORM** (not raw SQL) for database interactions.
- **Rule 1.4**: Apply **JWT authentication** for protected routes.
- **Rule 1.5**: Document all API endpoints using **Swagger (Flasgger)** annotations.

### **Database & Models**
- **Rule 1.6**: Define models in `models/` with proper SQLAlchemy column types.
- **Rule 1.7**: Use **`db_session`** for transactions (`add`, `commit`, `rollback` on errors). Prefer SQLAlchemy 2.0 style (`db_session.get()`) for new code.
- **Rule 1.8**: Avoid business logic in models; keep them data-only.

### **Error Handling & Logging**
- **Rule 1.9**: Use `@app.errorhandler` for consistent error responses (JSON format).
- **Rule 1.10**: Log errors and critical events via **`backend/config/logging_config.py`** (`setup_logging(app)`).

### **Testing**
- **Rule 1.11**: Write **Pytest** unit tests for all API endpoints.
- **Rule 1.12**: Use in-memory SQLite integration tests; mock only external HTTP/APIs (not the local DB).
- **Rule 1.13**: Module docstring on every `.py` file; Google-style docstrings and type hints on public functions/classes. See [documentation.mdc](documentation.mdc) and `backend/AGENTS.md`.

See `backend/AGENTS.md` for extension recipes.

---
> Source: [Elizio/vanilla-webapp-framework](https://github.com/Elizio/vanilla-webapp-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
