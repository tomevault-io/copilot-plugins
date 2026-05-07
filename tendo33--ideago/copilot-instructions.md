## python-configuration-management

> Best practices for configuration management using Pydantic Settings with environment variables and .env files.

- Use pydantic-settings for type-safe configuration management.
- Define all configuration in a Settings class inheriting from BaseSettings.
- Use Field() with default values and descriptions for all configuration fields.
- Support environment variables with clear naming conventions (UPPERCASE_WITH_UNDERSCORES).
- Use .env files for local development; never commit sensitive data.
- Implement get_settings() as a singleton pattern using @lru_cache.
- Provide helper methods in Settings class for derived paths and values.
- Use env_prefix for namespacing environment variables if needed.
- Validate configuration values using Pydantic validators.
- Document all configuration fields with Field(description=...).
- Support multiple environments (development, staging, production) with environment variable.
- Use model_config for Pydantic v2 settings like env_file, env_nested_delimiter.

---
> Source: [Tendo33/IdeaGo](https://github.com/Tendo33/IdeaGo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
