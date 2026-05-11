## pyscript-review

> **Project**: PuePy/PyScript SPA with Basecoat UI for peer review. Client-side only, German 1.0-5.0 grading. No backend, no database, no shadcn/React.

# pyscript_review — Copilot Instructions

**Project**: PuePy/PyScript SPA with Basecoat UI for peer review. Client-side only, German 1.0-5.0 grading. No backend, no database, no shadcn/React.

Full project context: `AGENTS.md`. Below are the 12 most critical rules.

---

## 1. Component Instantiation — `t.*()` Builder ONLY

**This is the #1 source of runtime bugs.** PuePy docs: "Components should not be created directly."

- **REQUIRED**: `t.sci_pro_card(variant="outlined")` — Builder provides `ref`, `origin`, `page` automatically.
- **BANNED**: `SciProCard(variant="outlined")` — crashes with `Tag.__init__() missing 'ref'`.
- **BANNED**: Importing component classes into pages. Use `t` Builder only.

```python
# ✅ CORRECT
with t.sci_pro_card(variant="outlined"):
    t.p("Content")

# ❌ WRONG — CRASHES
from src.components.atomic.card import SciProCard
SciProCard(variant="outlined")
```

Naming: class `SciProCard` → `component_name = "sci-pro-card"` → `t.sci_pro_card()`.

## 2. Two-State Model

- `self.state` = UI-local (tabs, accordions, drag flags)
- `self.application.state` = all business data (mode, session, criteria, grades)
- `self.state.get("key")` does NOT fall through — use the correct one

## 3. `bind=` Leak Fix

Every component with `bind=...` needs:
- `self._bind_key = kwargs.pop("bind", "")` in `__init__`
- `_handle_bind(self, kwargs): kwargs.pop("bind", None); self.bind = None`

Applies to: `SciProInput`, `SciProSelect`, `SciProTextarea` (and any future bind-using component).

## 4. No `self.update()`

`Component.update()` does NOT exist in PuePy. Use `self.page.redraw_tag(self)` or mutate reactive keys.

## 5. `create_proxy` for JS Events

`window.addEventListener` requires `pyodide.ffi.create_proxy`. Remember to call `.destroy()` in `unbind()`.

## 6. No Mutable Class Defaults

`list`, `dict`, `set` as class attributes are shared across instances. Use `None` + init in `bind(): self._items = []`.

## 7. Basecoat, Not shadcn

CSS classes: `.btn`, `.card`, `.input`, `.table`. No npm, no npx, no React/shadcn/Next.js. Components are Python `@t.component()` classes.

## 8. BasecoatBridge on Transitions

Call `window.basecoat.initAll()` after every PuePy page transition/redraw to re-init Basecoat JS (modals, dropdowns, tabs).

## 9. IndexedDB for Reviews

localStorage = 5MB limit. Reviews go in IndexedDB via `src/browser/` JS interop. localStorage only for small config.

## 10. ALL State Keys in `DEFAULT_STATE`

Every app state key must be declared in `src/state.py:DEFAULT_STATE` before use. No dynamic key injection.

## 11. Component Slots, Props, and Events

- **Props**: `props = ["variant", Prop("size", "Card size", str, "md")]`
- **Events**: `self.trigger_event("name", detail={...})` → consume with `on_name=self.handler`
- **Slots**: Define with `self.insert_slot("slot-name")`. Consume with `component.slot("slot-name")`. **Never `t.slot()`**.
- **Parent/Child**: `self.page`, `self.origin`, `self.parent`. Do NOT modify these.

## 12. Code Quality Gate

- `ruff`: 100 chars, double quotes, Google docstrings
- `pyright`: basic, pythonVersion=3.13
- `pytest`: test paths in `tests/`
- Commands: `task check` (ruff+pyright), `task format` (ruff format), `task test` (pytest -v), `task ci` (both)

---
> Source: [EmKaCe/pyscript_review](https://github.com/EmKaCe/pyscript_review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
