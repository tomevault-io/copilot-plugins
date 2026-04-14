## temp-tt

> Use Clarity Design System components in Angular. Apply when editing components or templates that use Clarity.


# Clarity Design System Components

This project uses **Clarity Design System** ([clarity.design](https://clarity.design/)) with **Clarity Angular** ([ng-clarity](https://github.com/vmware-clarity/ng-clarity)). Prefer Clarity components over custom or raw HTML for UI.

## Package and imports

- Use `@clr/angular` for Angular components. The app also uses `@clr/ui` (styles) and `@clr/icons`.
- Import `ClarityModule` (or specific sub-modules) in standalone components via `imports: [ClarityModule, ...]`.
- For icons, ensure `ClarityIcons` is configured in the app (e.g. in `main.ts` or app bootstrap).

## Layout (structure)

Use Clarity layout classes for page structure:

- **Container:** `class="main-container"` for the root layout.
- **Header:** `class="header header-6"` with `.branding` and `.nav-link` / `.title`.
- **Content:** `class="content-container"` then `class="content-area"` for main content.

```html
<div class="main-container">
  <header class="header header-6">
    <div class="branding">
      <a href="..." class="nav-link"><span class="title">App Title</span></a>
    </div>
  </header>
  <div class="content-container">
    <div class="content-area">
      <!-- page content -->
    </div>
  </div>
</div>
```

## Components to prefer

| Use case        | Use Clarity                         | Avoid           |
|----------------|--------------------------------------|-----------------|
| Buttons        | `class="btn btn-primary"` (or `btn-secondary`, etc.) | Raw `<button>` without Clarity classes |
| Data tables    | `<clr-datagrid>`, `<clr-dg-column>`, `<clr-dg-row>`, `<clr-dg-cell>` | Raw `<table>` for list data |
| Icons          | `<clr-icon shape="info-circle">`     | Custom icon markup        |
| Alerts         | `class="alert alert-info"` with `.alert-items`, `.alert-item`, `.alert-text` | Ad-hoc alert divs |
| Forms          | Clarity form containers and `clrInput`, `clrSelect`, etc. | Plain inputs without Clarity |
| Modals         | Clarity modal components            | Custom modal divs         |
| Tabs           | Clarity tabs                        | Custom tab markup         |
| Cards          | Clarity card components            | Custom card divs           |

## Data grid

Use `<clr-datagrid>` for tabular/list data. Define columns, then rows with `*ngFor`:

```html
<clr-datagrid>
  <clr-dg-column>Name</clr-dg-column>
  <clr-dg-column>Date</clr-dg-column>
  <clr-dg-row *ngFor="let item of items">
    <clr-dg-cell>{{ item.name }}</clr-dg-cell>
    <clr-dg-cell>{{ item.date | date }}</clr-dg-cell>
  </clr-dg-row>
  <clr-dg-footer>{{ items.length }} item(s)</clr-dg-footer>
</clr-datagrid>
```

## Reference

- Docs: [clarity.design](https://clarity.design/)
- Angular components source: [github.com/vmware-clarity/ng-clarity](https://github.com/vmware-clarity/ng-clarity)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tsvetomirgit)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tsvetomirgit)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
