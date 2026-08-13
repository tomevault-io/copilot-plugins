## python-neural-network

> - Do not edit or remove existing comments or docstrings unless I explicitly ask in that turn.

# AGENTS.md
# Note that this isn't a complete guarantee and it is possible for the following directives to
# be ignored on occasion

## Comments And Docstrings
- Do not edit or remove existing comments or docstrings unless I explicitly ask in that turn.
- If a code change would make an existing comment or docstring inaccurate, stop and ask before editing it.
- This applies even to small wording, typo, style, or clarity changes.
- Do not add new code comments.
- Do not add new docstrings unless asked to do so.

## Code Changes
- Never edit or add code unless explicitly asked to do so.
- If a change I request requires consequential changes elsewhere, tell me rather than making them.

## Tests
- When referring to arrays in test method names, I prefer `2D` over `2d` and `3D` over `3d`, even if this is inconsistent with the snake_case convention.
- When adding empty methods, I prefer them to contain `pass` rather than `...`.
- When passing values within tensors to backend methods as part of tests use floats unless the test is specifically testing the normalisation/rejection of non-float values.
- When I express a preference for how tests should be named prioritise clarity over literal compliance.
- Do not add empty lines in test methods

---
> Source: [jimnarey/python-neural-network](https://github.com/jimnarey/python-neural-network) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
