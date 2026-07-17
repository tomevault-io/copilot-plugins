## modular-development

> Modular architecture conventions for the MyProSole Streamlit app


# Modular Development

New features and changes MUST follow the existing modular architecture in `myprosole_app/`.

- Implement each feature as a self-contained module under `modules/<feature>/` with an `__init__.py` that exposes a module class plus a top-level `register(registry)` function (see `modules/example_feature/`).
- The module class declares `id`, `display_name`, and an `order` (int controlling sidebar/render sequence), and implements `render(ctx)`. Use `register_sidebar(ctx)` for sidebar widgets and the optional `render_analysis_tab(ctx)` for post-analysis tab UI.
- Activate modules by adding their import path to `ENABLED_MODULES` in `config.py`. Never hardcode module wiring elsewhere.
- Consume shared upload/data through `AppContext` params (e.g. `ctx.param("shared_data")`), never re-upload or re-load the file inside a module. Publish results for others via `ctx.set_param(...)`.
- Avoid cross-module tight coupling: discover and interact with other features through the registry (`ModuleRegistry`, `render_analysis_tab` protocol), not direct imports of sibling modules.
- Keep `core/` generic and feature-agnostic (`core/context.py`, `core/registry.py`, `core/loader.py`, `core/bootstrap.py`, `core/domain/`). Feature-specific logic lives inside its module.
- Prefer small, composable functions. Separate domain logic (`core/domain/`, `myprosole_analysis/`) from Streamlit/UI rendering so logic stays testable.

```python
# modules/my_feature/__init__.py
class MyFeatureModule:
    id = "my_feature"
    display_name = "My Feature"
    order = 50
    def render(self, ctx):
        data = ctx.param("shared_data")  # reuse shared upload
        ...

def register(registry):
    registry.add(MyFeatureModule())
```

---
> Source: [MyProEye-UG/MyProSole](https://github.com/MyProEye-UG/MyProSole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
