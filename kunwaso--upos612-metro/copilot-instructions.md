## projectx-module-ui

> Metronic 8.3.3 UI rules for the ProjectX module — assets, components, and markup (project-wide theme)


# ProjectX Module — UI & Design Rules (Metronic 8.3.3)

This rule applies to files inside `Modules/ProjectX/`. The **entire project** uses **Metronic 8.3.3**; core Blade views follow the same theme (see `ai/ui-components.md` and AGENTS.md Section 10).

## Theme: Metronic 8.3.3 (Bootstrap 5)

ProjectX (and the rest of the project) uses Metronic 8.3.3. ProjectX assets are published to `public/modules/projectx/`.

## Asset Paths

All CSS, JS, images, icons, and media must come from the ProjectX asset folder:

```
Source:    Modules/ProjectX/Resources/assets/
Published: public/modules/projectx/
```

In Blade, always reference assets with:

```blade
{{ asset('modules/projectx/...') }}
```

**Common paths:**

| Asset | Blade Path |
|-------|-----------|
| Main CSS | `asset('modules/projectx/css/style.bundle.css')` |
| Global plugins CSS | `asset('modules/projectx/plugins/global/plugins.bundle.css')` |
| Global plugins JS | `asset('modules/projectx/plugins/global/plugins.bundle.js')` |
| Scripts JS | `asset('modules/projectx/js/scripts.bundle.js')` |
| DataTables | `asset('modules/projectx/plugins/custom/datatables/datatables.bundle.{css,js}')` |
| Images/SVGs | `asset('modules/projectx/media/...')` |
| Icons | Keenicons via `<i class="ki-duotone ki-{name} fs-{size}">` |

**Never:**
- Use `asset('modules/projectx/assets/...')` — no nested `assets/` folder exists
- Invent new CSS classes — only use Metronic classes from the HTML reference

## HTML Component Reference

Before writing any Blade markup, find the matching component in:

```
Modules/ProjectX/Resources/html/
```

This folder contains Metronic reference pages organized by category:

| Need | Look in |
|------|---------|
| Page layout, sidebar, header | `html/layouts.html`, `html/asides.html`, `html/toolbars.html` |
| Dashboard cards/charts | `html/dashboards/*.html` |
| Data tables, listings | `html/apps/ecommerce/sales/listing.html`, `html/apps/customers/list.html`, `html/apps/user-management/users/list.html` |
| Forms, inputs, selects, editors | `html/forms/*.html`, `html/forms/editors/*.html` |
| Modals, popups | `html/utilities/modals/**/*.html` |
| Wizards, steppers | `html/utilities/wizards/*.html` |
| Cards, statistics, KPIs | `html/widgets/statistics.html`, `html/widgets/mixed.html`, `html/widgets/feeds.html` |
| User profiles, account pages | `html/account/*.html`, `html/pages/user-profile/*.html` |
| Authentication pages | `html/authentication/**/*.html` |
| Search overlays | `html/utilities/search/*.html` |
| Invoices | `html/apps/invoices/**/*.html` |
| Chat | `html/apps/chat/*.html` |
| File manager | `html/apps/file-manager/*.html` |
| Pricing tables | `html/pages/pricing/*.html` |
| Blog / CMS | `html/pages/blog/*.html` |
| FAQ | `html/pages/faq/*.html` |

### Workflow

1. **Identify** what UI you need (table, form, modal, card, widget, etc.)
2. **Open** the matching `html/` reference file from the table above
3. **Copy** the exact HTML structure, classes, and data attributes
4. **Adapt** to Blade: replace static text with `{{ }}`, add `@csrf`, wire routes
5. **Never** invent new CSS classes — use only what exists in `style.bundle.css`

## Class & Component Patterns

Use only Metronic/Bootstrap 5 classes found in the HTML references:

- **Cards**: `card`, `card-flush`, `card-body`, `card-header`, `card-title`, `card-toolbar`
- **Tables**: `table`, `table-row-dashed`, `table-row-gray-300`, `align-middle`, `gs-0`, `gy-4`
- **Buttons**: `btn`, `btn-primary`, `btn-light-primary`, `btn-sm`, `btn-icon`, `btn-hover-rise`
- **Badges**: `badge`, `badge-light-success`, `badge-light-danger`, `badge-light-warning`
- **Forms**: `form-control`, `form-select`, `form-check`, `form-label`, `form-control-solid`
- **Spacing**: `mb-5`, `mb-lg-10`, `pt-7`, `px-9`, `me-2`, `ms-3` (Bootstrap 5 utilities)
- **Text**: `text-gray-900`, `text-gray-700`, `text-muted`, `fw-bold`, `fw-semibold`, `fs-6`, `fs-7`
- **Grid**: `row`, `col-md-6`, `col-lg-4`, `col-xl-3`, `g-5`, `g-xl-10`
- **Icons**: `ki-duotone ki-{name}` with `<span class="path1"></span>...` children
- **Modals**: `modal`, `modal-dialog`, `modal-content`, `modal-header`, `modal-body`, `modal-footer`
- **Tabs**: `nav nav-tabs`, `nav-link`, `tab-content`, `tab-pane`
- **Alerts**: `alert`, `alert-dismissible`, `alert-primary`

## Date and Time Inputs

- **Do not use native HTML5 date/time widgets** in ProjectX views: no `type="date"`, `type="time"`, or `type="datetime-local"`.
- **Single date fields:** Use a text input with Metronic form classes and initialize Flatpickr in page JS. Keep submitted values in `Y-m-d` format.
- **Datetime fields:** Use a text input with Metronic form classes and initialize Flatpickr with time enabled. Keep submitted values in `Y-m-d\TH:i` format when the backend already expects that shape.
- **Date ranges:** Use Daterangepicker for true range controls, matching the Metronic `Resources/html` examples.
- Reuse existing page-level `@section('page_javascript')` blocks for picker init; do not create ad-hoc global assets for one-off form pages.

## Validation Before Finishing

Before declaring any ProjectX UI work done:

- [ ] Every asset path uses `asset('modules/projectx/...')` with no extra `assets/` segment
- [ ] Markup structure matches a reference in `Modules/ProjectX/Resources/html/`
- [ ] No custom CSS classes were invented — only existing Metronic classes
- [ ] Icons use `ki-duotone` with proper `<span class="pathN">` children
- [ ] DataTables use the Metronic DataTables plugin, not Yajra server-side rendering patterns from core UPOS
- [ ] No native `type="date"`, `type="time"`, or `type="datetime-local"` remain in ProjectX views; use Metronic Flatpickr or Daterangepicker instead

---
> Source: [kunwaso/upos612-metro](https://github.com/kunwaso/upos612-metro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
