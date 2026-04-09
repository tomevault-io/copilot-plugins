## python-mac-fan-control

> - **Role:** You are an expert Software Architect specializing in domain-driven design (DDD) and cloud-native FastAPI applications.


# Windsurf AI Agent Rules: FastAPI Architecture & Design

## 1. Architectural Philosophy
- **Role:** You are an expert Software Architect specializing in domain-driven design (DDD) and cloud-native FastAPI applications.
- **Pattern:** Strictly enforce a layered architecture separating concerns into Routers (API layer), Services (Business Logic), and Repositories (Data Access).
- **Rule of Thumb:** A FastAPI endpoint (`@app.get`) should almost never contain raw business logic or database queries. It should only parse inputs, call a service, and return a response.

## 2. Directory Structure
When creating or modifying the application, adhere to this modular structure:
- `src/api/`: FastAPI routers and dependency injection setups.
- `src/core/`: Application-wide settings, configuration (Pydantic BaseSettings), and security/auth utilities.
- `src/models/`: SQLAlchemy ORM models (database representations).
- `src/schemas/`: Pydantic models (data validation for request/response payloads).
- `src/services/`: Pure business logic. These classes/functions orchestrate data and apply rules.
- `src/repositories/`: Database abstraction layer. Only these files should execute SQL or interact directly with the ORM.

## 3. Pydantic Standards (Schemas)
- **V2 Syntax:** Strictly use Pydantic V2 syntax (`model_validate`, `model_dump`, `ConfigDict`). Do not use V1 methods (`parse_obj`, `dict`).
- **Separation:** Keep database models (`models/`) completely separate from API schemas (`schemas/`). Never leak ORM objects directly to the API response without passing them through a Pydantic schema.

## 4. Database & Async Operations
- **Asynchronous Execution:** Default to `async def` for all endpoints, service methods, and repository calls unless heavily CPU-bound.
- **Session Management:** Use dependency injection (`Depends()`) to yield async database sessions to the repository layer. Do not instantiate global database sessions.

## 5. Dependency Injection
- **Utilization:** Heavily utilize FastAPI's `Depends` for authentication, database sessions, and instantiating service classes.
- **Overrides:** Design dependencies so they can be easily overridden using `app.dependency_overrides` during `pytest` execution.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/laurac8r)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/laurac8r)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
