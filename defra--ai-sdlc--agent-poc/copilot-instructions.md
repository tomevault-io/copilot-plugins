## ai-sdlc-agent-poc

> Code Style for Python Projects

# Code Style for Python Projects

## Base Standards
- Follow PEP 8 conventions
- Use Ruff for linting and formatting
- Use Pyright for type checking

### Formatting
- Indent: 4 spaces
- Line Length: 88 chars
- Use double quotes for strings
- Docstring: Google style

## Linting Rules
Ruff is configured as per @ruff.yml

## Naming Conventions
- Use `snake_case` for functions, variables, and modules
- Use `PascalCase` for classes
- Use `UPPER_CASE` for constants

## Import Order
Imports are automatically sorted by Ruff using isort rules:
1. Standard library imports
2. Third-party imports (fastapi, pydantic, motor, pytest)
3. Local application imports

## Function Guidelines
- Maximum function length: 50 lines
- Maximum nesting depth: 3 levels
- Single responsibility principle
- Clear return types

## Automatic Formatting
- Format code on save with Ruff
- Fix linting issues automatically when possible
- Use VS Code with Ruff extension for best experience

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DEFRA) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
