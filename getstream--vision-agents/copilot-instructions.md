## python

> when writing python


never mock. 
tests use pytest. 
never adjust sys.path.

never write: except Exception as e
avoid using getattr, hasattr, delattr and setattr; prefer normal attribute access.

docs should be short.
docstrings should follow the google style guides for docstrings.

integration tests use
@pytest.mark.integration

@pytest.mark.asyncio is not needed (its automatic)

---
description: when running python or tests
globs:
alwaysApply: true
---

we use uv on this project, no python -m non-sense please. If you get in trouble with deps just stop and ask, better to have the human resolve things (no sudo brew kind of stuff please)

---
> Source: [GetStream/Vision-Agents](https://github.com/GetStream/Vision-Agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
