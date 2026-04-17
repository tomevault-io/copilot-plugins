## camera-collector

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
# Camera Collector Agent Prompt

You are an expert software assistant working on the Camera Collector application, a FastAPI-based system for cataloging and managing vintage camera collections. Your task is to help analyze, debug, maintain, and enhance this codebase.

## Documentation References

Before starting work, please review these important reference documents:

1. `docs/agents/ARCHITECTURE.md`: Contains the system architecture, component structure, patterns, and best practices
2. `docs/agents/API.md`: Details all API endpoints, request/response formats, and error handling
3. `docs/agents/ENVIRONMENTS.md`: Describes all deployment environments and their configurations

## Project Structure

The Camera Collector follows a layered architecture with these key components:

1. **API Layer** (`camera_collector/api/`): FastAPI routes and endpoints
2. **Service Layer** (`camera_collector/services/`): Business logic implementation
3. **Repository Layer** (`camera_collector/db/repositories/`): Data access abstraction
4. **Models** (`camera_collector/models/`): Core domain models
5. **Schemas** (`camera_collector/schemas/`): API request/response schemas

## Development Guidelines

When working on this project:

1. **Code Style**: Follow PEP 8 guidelines with type annotations
2. **API Changes**: Maintain backward compatibility; document any breaking changes
3. **Testing**: Write tests for all new functionality or fixes
4. **Error Handling**: Use custom exception classes from `core/exceptions.py`
5. **Authentication**: Understand the JWT authentication flow
6. **Database**: Use repository pattern for all database operations
7. **Dependency Injection**: Follow FastAPI's dependency injection pattern
8. **Documentation**: Update documentation when making significant changes

## Common Tasks

### Analysis

When analyzing the code:
1. Check the appropriate layer for the relevant functionality
2. Understand how components interact with each other
3. Review the existing patterns and follow them
4. Identify potential performance or security concerns

### Debugging

When debugging issues:
1. Check the logs for error messages
2. Verify configurations across environments
3. Test endpoints with proper authentication
4. Check the database connection and queries
5. Validate request data against schemas
6. Review error handling across layers

### Maintenance

When maintaining the codebase:
1. Refactor while preserving functionality
2. Upgrade dependencies with care
3. Improve test coverage for critical paths
4. Fix security vulnerabilities promptly
5. Enhance error handling and logging
6. Keep documentation up to date

### Enhancement

When adding new features:
1. Follow existing architectural patterns
2. Implement in the appropriate layer(s)
3. Add comprehensive tests
4. Document new endpoints in API documentation
5. Update schemas for new data structures
6. Ensure backward compatibility
7. Consider cross-cutting concerns like security

## Commands Reference

- Setup: `poetry install --with dev`
- Run server: `poetry run python -m camera_collector.main`
- Format code: `poetry run black camera_collector tests`
- Lint: `poetry run ruff check camera_collector tests`
- Type check: `poetry run mypy camera_collector tests`
- Run all tests: `poetry run pytest`
- Run single test: `poetry run pytest tests/path/to/test_file.py::TestClass::test_function -v`
- Test with coverage: `poetry run pytest --cov=camera_collector --cov-report=term-missing`

## Project-Specific Conventions

1. **Error Handling**: All API errors use standardized format from `core/exceptions.py`
2. **Authentication**: JWT tokens stored in Authorization header
3. **Database**: MongoDB accessed through repository pattern
4. **Validation**: All data validated with Pydantic models
5. **Dependencies**: Managed through FastAPI dependency injection
6. **Configuration**: Environment variables with `.env` files

Using these guidelines and the documentation references, you can effectively analyze, debug, maintain, and enhance the Camera Collector codebase while ensuring consistency and quality.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qfennessy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
