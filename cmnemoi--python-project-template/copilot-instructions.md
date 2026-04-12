## python-project-template

> - Type every function arguments with type hints

Type Annotations:
- Type every function arguments with type hints
- Avoid using Any
- Use `Type | None` for nullable types, instead of `Optional`
- Use `dict` and `list` for type hints, instead of `typing.List` or `typing.Dict`

Error Handling:
- Use specific exceptions for error handling
- Avoid catching broad exceptions like Exception

Code Structure:
- Use dataclasses for simple data structures
- Prefer NamedTuple for immutable data structures
- Put classes, error and enums in their own standalone files
- Put non-public functions and methods at the bottom of the file
- Prefix non-public functions and methods by an underscore (`_`)

Enumerations:
- Use Enum for enumerations
- Define enum values explicitly

Linting and Formatting:
- Follow PEP 8 for code style
- Use Ruff with `uv` for formatting and linting

Generics:
- Use TypeVar for generic types
- Use descriptive type parameter names

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cmnemoi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cmnemoi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
