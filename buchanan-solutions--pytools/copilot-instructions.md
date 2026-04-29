## pytools

> Procedures for adding new utility subpackages to pytools (e.g., fastapi, sql)


# Adding a New pytools Subpackage

When creating a new utility subpackage (e.g., `pytools.fastapi`), follow this exact 3-step workflow to maintain architectural integrity and dependency flow.

## 1. Directory Structure
Create a new directory under `src/pytools/`. Every subpackage must have an `__init__.py` to be discoverable.

**Action:**
- Create `src/pytools/<name>/`
- Create `src/pytools/<name>/__init__.py` (Keep this empty or for public exports only)

## 2. Update pyproject.toml
Modify `[project.optional-dependencies]` to include the new subpackage. 

**Requirements:**
- **Core Dependency:** Every new subpackage MUST include `"pytools[core]"` as the first item in its list.
- **External Deps:** List necessary third-party libraries (e.g., `fastapi`, `sqlalchemy`).
- **Update 'all':** Add the new extra to the `all` group.

**Example Pattern:**
```toml
[project.optional-dependencies]
fastapi = [
    "pytools[core]",
    "fastapi>=0.115.0",
]
all = [
    "pytools[typer,pydantic,debugging,fastapi]"
]

```

## 3. Code & Imports

When writing logic for the new subpackage, leverage the existing `core` layer for shared utilities.

**Standard Import Pattern:**

```python
from pytools.core.get_from_obj import get_from_obj

```

## 4. Verification

After adding a subpackage, remind the user to run:

1. `uv lock` to update dependencies.
2. `uv build` to ensure the Hatchling backend correctly identifies the new folder.

---

### Why this works well in Cursor:
* **Context Awareness:** Because of the `globs`, the AI will "remember" these instructions specifically when you are editing `pyproject.toml` or adding files in the `src` directory.
* **Hatchling Support:** It reinforces the `src` layout we established, preventing the AI from accidentally creating folders in the root directory.
* **Dependency Safety:** By mandating `"pytools[core]"`, it ensures that even as you scale to 10+ subpackages, your foundational logic remains available everywhere.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/buchanan-solutions) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
