## api-patterns

> - Use FastAPI dependency injection for shared resources

# API Development Patterns

- Use FastAPI dependency injection for shared resources
- Define Pydantic models for all request and response bodies
- Use async def for all endpoint handlers
- Use the `validate_access_token` middleware (automatically added to all endpoints) for authentication
- Return appropriate HTTP status codes (201 for creation, 204 for deletion, etc.)
- Include proper OpenAPI documentation in endpoints

---
> Source: [METR/inspect-action](https://github.com/METR/inspect-action) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
