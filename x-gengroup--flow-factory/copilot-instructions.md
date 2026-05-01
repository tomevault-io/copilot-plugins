## no-section-divider-comments

> Do not add section-divider comments (e.g., # --- Section ---). Use descriptive docstrings and clear function/class names instead.


Avoid decorative section-divider comments in Python files. Prefer:
- Module-level docstrings for file purpose
- Class/function docstrings for component documentation
- Inline comments only for non-obvious logic

Bad:
```python
# ========================
# Model Loading Section
# ========================
```

Good:
```python
class ModelLoader:
    """Handles model loading and initialization."""
```

---
> Source: [X-GenGroup/Flow-Factory](https://github.com/X-GenGroup/Flow-Factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
