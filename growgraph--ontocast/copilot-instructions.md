## pydantic-usage

> Conventions for using Pydantic models and settings


# Pydantic Rules
- Use `pydantic.BaseModel` for request/response schemas.
- For configuration, use `pydantic_settings.BaseSettings` (v2).
- Always provide default values or use `...` for required fields.
- Use model validators (`@field_validator`, `@model_validator`) instead of custom init logic.
- Keep models small and focused on one purpose.

---
> Source: [growgraph/ontocast](https://github.com/growgraph/ontocast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
