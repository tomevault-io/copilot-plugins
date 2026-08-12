## code-quality

> Code quality standards for ecommerce_recommendator


# Code Quality Standards

## Linting Rules

- **NEVER use inline `noqa` comments** to suppress linter warnings
- Fix the underlying issue instead of ignoring it
- For project-wide patterns (like Django settings), configure exceptions in `pyproject.toml` under `[tool.ruff.lint.per-file-ignores]`

## Examples

```python
# ❌ BAD - Inline suppression
def health_check(request: object) -> JsonResponse:  # noqa: ARG001
    pass

# ✅ GOOD - Use underscore prefix for intentionally unused args
def health_check(_request: object) -> JsonResponse:
    pass
```

```python
# ❌ BAD - Inline noqa
from .base import *  # noqa: F403

# ✅ GOOD - Configure in pyproject.toml per-file-ignores for Django patterns
# In pyproject.toml:
# [tool.ruff.lint.per-file-ignores]
# "core/settings/*.py" = ["F403", "F405"]
```

## Type Checking

- Use strict mypy configuration
- Avoid `# type: ignore` comments - fix the typing issue instead
- **Exception**: Known bugs in external libraries (e.g., django-stubs issues) may use `type: ignore[specific-code]` WITH a comment linking to the issue
- Prefer proper typing over ignoring errors

```python
# ❌ BAD - Unexplained type ignore
class UserAdmin(BaseUserAdmin):  # type: ignore

# ✅ GOOD - Documented known issue with link
# UserAdmin is generic in django-stubs but not subscriptable at runtime
# See: https://github.com/typeddjango/django-stubs/issues/1097
class UserAdmin(BaseUserAdmin):  # type: ignore[type-arg]
```

---
> Source: [Plenix-AI/ecommerce_recommendator](https://github.com/Plenix-AI/ecommerce_recommendator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
