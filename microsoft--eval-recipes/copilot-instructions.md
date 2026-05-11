## eval-recipes

> - This is a production-grade Python package using `uv` as the package and project manager. You must *always* follow best open-source Python practices.

# General Instructions
- This is a production-grade Python package using `uv` as the package and project manager. You must *always* follow best open-source Python practices.
- Shortcuts are not appropriate. When in doubt, you must work with the user for guidance.
- Any documentation you write, including in the README.md, should be clear, concise, and accurate like the official documentation of other production-grade Python packages.
- Make sure any comments in code are necessary. A necessary comment captures intent that cannot be encoded in names, types, or structure. Comments should be reserved for the "why", only used to record rationale, trade-offs, links to specs/papers, or non-obvious domain insights. They should add signal that code cannot.
- The current code in the package should be treated as an example of high quality code. Make sure to follow its style and tackle issues in similar ways where appropriate.
- Do not run tests automatically unless asked since they take a while.
- Don't generate characters that a user could not type on a standard keyboard like fancy arrows.
- Anything is possible. Do not blame external factors after something doesn't work on the first try. Instead, investigate and test assumptions through debugging through first principles.
- When writing documentation
  - Keep it very concise
  - No emojis or em dashes.
  - Documentation should be written exactly like it is for production-grade, polished, open-source Python packages.
- You should only use the dependencies in the provided dependency files. If you need to add a new one, ask first.
- Never add back in code or comments that someone else has removed or changed since you last viewed it.

# Python Development Instructions
- `ty` by Astral is used for type checking. Always add appropriate type hints such that the code would pass ty's type check.
- Follow the Google Python Style Guide.
- After each code change, checks are automatically run. Fix any issues that arise.
- **IMPORTANT**: The checks will remove any unused imports after you make an edit to a file. So if you need to use a new import, be sure to use it FIRST (or do your edits at the same time) or else it will be automatically removed. DO NOT use local imports to get around this.
- At this stage of the project, NEVER add imports to __init__.py files. Leave them empty unless absolutely necessary.
- Always prefer pathlib for dealing with files. Use `Path.open` instead of `open`.
- When using pathlib, **always** Use `.parents[i]` syntax to go up directories instead of using `.parent` multiple times.
- When writing tests, use pytest and pytest-asyncio.
  - The pyproject.toml is already configured so you do not need to add the `@pytest.mark.asyncio` decorator.
- Prefer using loguru for logging instead of the built-in logging module. Do not add logging unless requested.
- NEVER use `# type: ignore`. It is better to leave the issue and have the user work with you to fix it.
- Don't put types in quotes unless it is absolutely necessary to avoid circular imports and forward references.
- When writing multi-line strings, use `"""` instead of using string concatenation. Use `\` to break up long lines in appropriate places.

- When constructing long strings like prompts for LLMs, use `python-liquid`'s `render` function:
```python
from liquid import render

print(render("Hello, {{ you }}!", you="World"))
# Hello, World!
```
- Include `# Copyright (c) Microsoft. All rights reserved` at the top of each Python file (don't worry about __init__.py files)
- For Windows compatibility, use encoding='utf-8' when handling files, unless required otherwise.
- Leave __init__.py files empty unless there is a specific reason to add code there during this early stage of the project.
- When running scripts with `uv run` make sure you are in the top level of this Python project. All you need to do is `uv run <script relative path>`. You also do not need to do `uv run python`, the `python` is unnecessary.

# Key Files

@README.md

@pyproject.toml

@docs/BENCHMARKING.md

@src/eval_recipes/benchmarking/schemas.py

---
> Source: [microsoft/eval-recipes](https://github.com/microsoft/eval-recipes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
