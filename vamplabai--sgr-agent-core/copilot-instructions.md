## python-fastapi

> Python and FastAPI specific guidelines


# Python/FastAPI Guidelines

## General Python Rules

- Don't use the `requests` library in asynchronous code - use `httpx` instead
- Don't use `print` in the main logic of system services - initialize `logging.Logger`
- Use `def` for pure functions and `async def` for asynchronous operations
- Use type hints for all function signatures
- Prefer Pydantic models over raw dictionaries for input validation
- Write comments only in entity docs and complex logic - no need to comment every line of code
- Write comments only in English
- Do not use `Optional` import from `typing` in cases you can use `|` (Python 3.10+)

## Error Handling and Validation

- Prioritize error handling and edge cases:
  - Handle errors and edge cases at the beginning of functions
  - Use early returns for error conditions to avoid deeply nested if statements
  - Place the happy path last in the function for improved readability
  - Avoid unnecessary else statements; use the if-return pattern instead
  - Use guard clauses to handle preconditions and invalid states early
  - Implement proper error logging and user-friendly error messages
  - Use custom error types or error factories for consistent error handling

## FastAPI-Specific Guidelines

- Avoid global scope variables - implement/use global application state, set its initialization logic in one place
- Use functional components (plain functions) and Pydantic models for input validation and response schemas
- Use declarative route definitions with clear return type annotations
- Use `async def` for asynchronous endpoints
- Minimize `@app.on_event("startup")` and `@app.on_event("shutdown")` - prefer lifespan context managers for managing startup and shutdown events
- Use middleware for logging, error monitoring, and performance optimization
- Optimize for performance using async functions for I/O-bound tasks, caching strategies, and lazy loading
- Use `HTTPException` for expected errors and model them as specific HTTP responses
- Use middleware for handling unexpected errors, logging, and error monitoring
- Use Pydantic's `BaseModel` for consistent input/output validation and response schemas

## Performance Optimization

- Minimize blocking I/O operations - use asynchronous operations for all database calls and external API requests
- Implement caching for static and frequently accessed data using tools like Redis or in-memory stores
- Optimize data serialization and deserialization with Pydantic
- Use lazy loading techniques for large datasets and substantial API responses

## References

@code-style.mdc
@architecture.mdc
@core-modules.mdc

---
> Source: [vamplabAI/sgr-agent-core](https://github.com/vamplabAI/sgr-agent-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
