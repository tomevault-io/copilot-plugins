## blade-ui-style-enforcement

> Enforce Metronic 8.3.3 UI patterns in core Blade views


# Blade UI Style Enforcement (Core)

- This rule applies to **core** Blade views under `resources/views/`. The **entire project** uses **Metronic 8.3.3** (Bootstrap 5).
- For **new or migrated core views**, use Metronic patterns from **`public/html/`** (HTML reference) and **`public/assets/`** (assets). In Blade use `asset('assets/...')` for CSS, JS, media (see `ai/ui-components.md` and AGENTS.md Section 10).
- For **legacy core views** that still use Bootstrap/AdminLTE or other old classes: when touching them, migrate to Metronic or preserve only if the task explicitly excludes migration.
- Use only existing Metronic/Bootstrap 5 classes; do not invent new classes.
- Match component structure for cards, forms, tables, widgets, nav, sidebar, and modals to the Metronic HTML in `public/html/`.

## Styling Constraints

- Avoid inline styles in Blade unless explicitly requested.
- Do not create parallel design systems or UI kits.
- Core layouts load Metronic from `public/assets/` via `asset('assets/...')`.

## Validation Before Finishing

- Confirm a matching reference exists in `public/html/` for new or changed markup.
- Confirm Blade markup uses only Metronic/Bootstrap 5 class patterns from `ai/ui-components.md`.
- Confirm asset paths use `asset('assets/...')` and no invented or legacy theme classes were introduced.

---
> Source: [kunwaso/upos612-metro](https://github.com/kunwaso/upos612-metro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
