## code-style

> Code style and formatting rules for SGR Agent Core


# Code Style Rules

## General Rules

1. **Comments**: Write all code comments **ONLY in English**
2. **User responses**: Respond in Russian unless user requests otherwise
3. **Empty line at end of file**: Always add an empty line at the end of new files
4. **Virtual environment**: Virtual environment is located in `.venv` directory

## Code Formatting

### Ruff and Black
- Use `ruff` for linting and formatting
- Line length: **120 characters** (configured in `pyproject.toml`)
- Follow rules from `pyproject.toml` and `.ruff.toml` if present

### Imports
- Use `isort` for import sorting
- Group imports: standard library → third-party → local
- One import per line for long lists

### Type Hints
- **Mandatory**: Use type hints for all functions and methods
- Prefer `T | None` over `Optional[T]` (Python 3.10+)
- Use `Union[A, B]` for complex types when needed
- Use `dict[str, Any]` instead of `Dict[str, Any]` (Python 3.9+)

### Docstrings
- Use docstrings in Google style format
- All docstrings in **English**
- Must document:
  - Public classes and methods
  - Complex business logic
  - Parameters and return values

Example:
```python
def create_agent(self, agent_def: AgentDefinition, task_messages: list[dict]) -> BaseAgent:
    """
    Create an agent instance from a definition.

    Args:
        agent_def: Agent definition with configuration
        task_messages: Task messages in OpenAI format

    Returns:
        Created agent instance
    """
```

## File Structure

### Element Order in File
1. Module docstring
2. Imports (standard library → third-party → local)
3. Constants
4. Types and exceptions
5. Classes and functions

### Naming
- **Classes**: PascalCase (`SGRAgent`, `BaseTool`)
- **Functions and methods**: snake_case (`create_agent`, `_prepare_context`)
- **Constants**: UPPER_SNAKE_CASE (`MAX_ITERATIONS`)
- **Private methods**: start with `_` (`_reasoning_phase`, `_log_reasoning`)
- **Types**: PascalCase (`AgentContext`, `AgentDefinition`)

## Error Handling

- Use specific exceptions, not generic `Exception`
- Create custom exceptions for business logic when needed
- Always include informative error messages
- Use `Optional` or `| None` for values that may be missing

## Async/Await

- Use `async def` for all asynchronous operations
- Use `await` for all async calls
- Don't use blocking I/O in async code
- Use `httpx` instead of `requests` for async HTTP calls

## FastAPI-Specific Guidelines

- Avoid global scope variables, use application state
- Use functional components and Pydantic models for validation
- Use declarative route definitions with clear return type annotations
- Use `async def` for asynchronous endpoints
- Use Pydantic's `BaseModel` for input/output validation
- Use `HTTPException` for expected errors

## References

@pyproject.toml
@pytest.ini

---
> Source: [vamplabAI/sgr-agent-core](https://github.com/vamplabAI/sgr-agent-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
