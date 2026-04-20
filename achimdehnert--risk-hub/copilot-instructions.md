## risk-hub

> Every reusable UI block is a **Component** with 3 access paths:


# Django Component Pattern (ADR-041)

## Architecture
Every reusable UI block is a **Component** with 3 access paths:
1. **Inclusion Tag**: `{% load <app>_components %}` → `{% <name> obj %}`
2. **HTMX Fragment**: `hx-get="{% url '<app>-components:<name>' pk=obj.pk %}"`
3. **Template Include**: `{% include "<app>/components/_<name>.html" %}`

## File Structure (risk-hub: src/ prefix, bare module names)
```
src/<app>/
├── components/               # Component modules
│   ├── __init__.py           # Registry + exports
│   └── <name>.py             # get_context() + fragment_view()
├── templatetags/
│   └── <app>_components.py   # @register.inclusion_tag
└── urls_components.py        # HTMX fragment endpoints

src/templates/<app>/components/
├── _<name>.html              # Default variant (underscore prefix!)
├── _<name>_compact.html      # Compact variant (optional)
└── _<name>_card.html         # Card variant (optional)
```

## Component Anatomy
```python
# src/<app>/components/<name>.py
TEMPLATES: dict[str, str] = {
    "default": "<app>/components/_<name>.html",
    "compact": "<app>/components/_<name>_compact.html",
}

def get_context(obj, user, *, variant="default") -> dict:
    """Single source of truth for component data."""
    ...
    return {"obj": obj, "variant": variant, "template_name": TEMPLATES[variant]}

def fragment_view(request, pk) -> TemplateResponse:
    """HTMX fragment endpoint for lazy-loading."""
    ...
```

## Rules
- Every component MUST have `get_context()` as single data source
- Tag, View, and Tests all call the same `get_context()`
- Templates use underscore prefix: `_<name>.html`
- Max 3 variants per component: default, compact, card
- All fragment views: `@login_required`
- Use `select_related`/`prefetch_related` in `get_context()` (avoid N+1)
- `data-testid="<component>-<element>"` on all interactive elements
- Component must be used in ≥2 places to justify extraction
- CRITICAL: All queries in `get_context()` MUST filter by `tenant_id`

## HTMX Detection (risk-hub: django_htmx)
- Use `request.htmx` (django_htmx is installed)
- Fragment views return TemplateResponse (no full page)

## Existing Components (DSB — PILOT)
- `dsb_components`: `stat_card`, `status_badge`, `empty_state`
- Used in: dashboard.html, vvt_list.html, tom_list.html, dpa_list.html
- Phase 1 target: `data_table`, `detail_card`, `alert_banner`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/achimdehnert) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
