## nabla-memo

> This rule explains FastAPI conventions and best practices for high-performance Python APIs.


# FastAPI rules

- Use type hints for all function parameters and return values
- Use Pydantic models for request and response validation
- Use appropriate HTTP methods with path operation decorators (@app.get, @app.post, etc.)
- Use dependency injection for shared logic like database connections and authentication
- Use background tasks for non-blocking operations
- Use proper status codes for responses (201 for creation, 404 for not found, etc.)
- Use APIRouter for organizing routes by feature or resource
- Use path parameters, query parameters, and request bodies appropriately

---
> Source: [AlbanAndrieu/nabla-memo](https://github.com/AlbanAndrieu/nabla-memo) — distributed by [TomeVault](https://tomevault.io/claim/AlbanAndrieu).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
