## claudetainer

> Inspect project files to determine correct test, linting, or formatting options.

# Python Development Standards

## Language-Specific Commands
Inspect project files to determine correct test, linting, or formatting options.

## Code Style Preferences
- **Style**: Black formatting, snake_case naming
- **Type Hints**: Use for all function signatures
- **Models**: Pydantic models over raw dictionaries
- **Functions**: Prefer `def` for pure functions, `async def` for async operations
- **Imports**: Absolute imports, group by standard/third-party/local
- **Documentation**: Google-style docstrings for public functions

## Python Workflow Notes
- Use functional, declarative programming where possible
- Apply RORO pattern (Receive Object, Return Object)
- Handle errors early with guard clauses and early returns
- Use descriptive variable names with auxiliary verbs (is_active, has_permission)
- Run single tests frequently with pytest markers
- Use virtual environments for dependency isolation

---
> Source: [smithclay/claudetainer](https://github.com/smithclay/claudetainer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
