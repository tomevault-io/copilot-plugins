## arcade-toolkit-standards

> - Always use `@tool` decorator from `arcade_tdk`


# Arcade.dev Toolkit Development Standards

## Tool Implementation Patterns

### Tool Decoration and Typing
- Always use `@tool` decorator from `arcade_tdk`
- Use `requires_secrets=["SECRET_NAME"]` for tools needing secrets
- Implement comprehensive type hints with `Annotated` for parameters
- Use `ToolContext` for secret management: `context.get_secret("SECRET_NAME")`

### Error Handling Strategy
- Use `RetryableToolError` for user input validation errors (user can fix)
- Use `RuntimeError` for system/configuration errors (caught by @tool decorator)
- Always validate input parameters with clear error messages
- Example from [get_audio_list.py](mdc:foundaudio/foundaudio/tools/get_audio_list.py):
```python
if limit is not None and (limit < 1 or limit > 100):
    raise RetryableToolError(
        "Invalid limit parameter. Please provide a limit between 1 and 100.",
        additional_prompt_content="The limit parameter must be between 1 and 100."
    )
```

### Data Models and Validation
- Use Pydantic `BaseModel` for structured data (see `AudioFile` class)
- Validate all external API responses through Pydantic models
- Return dictionaries from tools (use `model_dump()` for Pydantic objects)

### Documentation Requirements
- Comprehensive docstrings explaining tool purpose, parameters, returns, and exceptions
- Verbose inline comments explaining business logic and API interactions
- Parameter descriptions using `Annotated[Type, "Description"]` pattern

### Secret Management
- Always check if secrets exist: `if not supabase_key: raise RuntimeError(...)`
- Use environment variables with defaults for configuration
- Document that some "secrets" may be public (like Supabase anon keys) but follow patterns

---
> Source: [rsmets/arcade-foundaudio-toolkit](https://github.com/rsmets/arcade-foundaudio-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
