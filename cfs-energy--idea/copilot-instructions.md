## idea

> - **AppContext Pattern**: Each module has an AppContext for dependency injection

# Python Development Guidelines for IDEA

## IDEA Python Architecture
- **AppContext Pattern**: Each module has an AppContext for dependency injection
- **API Invoker Pattern**: Route API requests through namespace-based invokers (`Module.Action`)
- **Data Model Pattern**: Use Pydantic models for validation (SocaBaseModel)

## Core Modules
- **ideadatamodel**: Shared Pydantic models and exceptions
- **ideasdk**: Common utilities and abstractions
- Module-specific packages (ideascheduler, ideavirtualdesktopcontroller, etc.)

## Key Patterns
- Use `context.config().get_string()` for configuration access
- Use `context.aws().service_name()` for AWS client access
- Use `SocaException` with `errorcodes` for error handling
- Use `ApiInvocationContext` for API request handling

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cfs-energy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
