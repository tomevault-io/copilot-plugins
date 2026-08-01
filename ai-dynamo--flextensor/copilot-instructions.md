## flextensor

> Follow `docs/api/glossary.md` for canonical definitions (trap vs module vs layer, weight vs parameter vs tensor).

<!--
# SPDX-FileCopyrightText: Copyright (c) 2025-2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: Apache-2.0
-->
# FlexTensor Source Guidelines

## Terminology (Docstrings)

Follow `docs/api/glossary.md` for canonical definitions (trap vs module vs layer, weight vs parameter vs tensor).

## Deprecation

**Policy**: Deprecated in `vX.Y` → remove in `vX.(Y+1)` minimum. State removal version explicitly.

Use `@deprecated(msg)` from `typing_extensions` (PEP 702). Add `.. deprecated:: vX.Y` to the docstring (`v`-prefix to match git tags).

- Stack `@deprecated` above `@property` (outermost)
- No replacement? Omit "Use X instead."
- Custom runtime: `@deprecated(msg, category=None)` + `warnings.warn(..., stacklevel=2)`

### Pydantic Fields

Combine `Annotated` + `Field(deprecated=...)` + model validator for construction-time warning:

```python
_MY_FIELD_MSG = "`old_field` is deprecated. Use `new_field` instead. Will be removed in vX.Y."

class MyConfig(BaseModel):
    old_field: Annotated[bool, _deprecated(_MY_FIELD_MSG)] = Field(default=False, deprecated=_MY_FIELD_MSG)

    @model_validator(mode="before")
    @classmethod
    def _sync_fields(cls, data):
        if isinstance(data, dict) and "old_field" in data and "new_field" not in data:
            warnings.warn(_MY_FIELD_MSG, DeprecationWarning, stacklevel=2)
            data["new_field"] = data["old_field"]
        return data
```

### Process

Commit: `chore(deprecate): mark <X> deprecated, remove in v<Y.Z>` → add to `### Deprecated` in `CHANGELOG.md`. Removal: `BREAKING CHANGE:` footer → `### Removed`.

---
> Source: [ai-dynamo/flextensor](https://github.com/ai-dynamo/flextensor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
