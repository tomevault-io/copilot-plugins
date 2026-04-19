## opensteuerauszug

> name: python_dependency_management

 # Cursor Rule: Python Dependency Management

<rule>
name: python_dependency_management
description: Instructions for managing Python dependencies

# Rule Content
Always add new Python dependencies to the appropriate section (`[project.dependencies]` or `[project.optional-dependencies]`) in the `pyproject.toml` file.

Avoid using `requirements.txt` for dependency management in this project.

Ensure dependencies are added under the `[project]` table.

Example (`pyproject.toml`):
```toml
[project]
# ... other project metadata ...
dependencies = [
    "pydantic",
    "typer",
    "rich",
    "new_dependency_here", # Add new dependencies here
]

[project.optional-dependencies]
development = [
    "pytest", # Add dev dependencies here
]
```

</rule>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vroonhof) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
