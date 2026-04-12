## discount-analyst

> - Always use best practices when writing Python code.


# Python guidelines

When writing Python:

- Always use best practices when writing Python code.
- Prefer named attribute access (Pydantic models, dataclasses, NamedTuples) over plain tuples or dicts.
- Always import from the top of the file when possible.
- I am using Python 3.14 in this repo. The latest Python 3.14 syntax is preferred.
- Do not create barrel import files or use __all__ in __init__.py files. it is an antipattern as it makes circular dependencies more common.
- Always use the rich library for pretty printing when writing Python scripts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Samuel-Harris)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Samuel-Harris)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
