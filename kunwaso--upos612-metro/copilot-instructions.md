## ui-layout-first-then-data

> For UI/style building — first match layout structure from reference (no missing pieces), then wire controller data


# UI Build Order — Layout First, Then Data

When building or rebuilding a page to match a **reference template** (e.g. from `public/html/account/`, `public/html/`, or a Metronic HTML file):

## 1. First — Layout match (structure only)

- **Copy and adapt** the reference HTML into Blade so the **layout and structure** are complete and match the reference.
- Use **static titles, placeholder content, and placeholder actions** so the page renders with the correct look and no missing sections (toolbar, cards, tabs, panes, nav, etc.).
- **Do not** depend on controller variables or form actions at this stage. The page should look right and interactive structure (e.g. tab switching) should work with placeholders.
- Ensure: no missing layout pieces, correct Metronic/theme classes, correct DOM structure. Verify by opening the page and checking that the layout matches the reference.

## 2. Then — Add controller data

- **After** the layout is in place, wire in controller data: form `action`, `method`, `id`; submit button; any `@include` partials; and inside partials, controller variables and `Form::*` (or equivalent) so every field has real data and the form submits correctly.
- Do **not** change the template structure (classes, layout) when wiring data — only bind values, names, and actions.
- All view data must come from the controller (or Util/composer); no business logic or variable defaulting in Blade (see Laravel coding constitution).

## Why this order

- Reduces mistakes: template and data wiring are separate steps, so agents do not mix structure edits with data binding.
- Easier to verify: you can confirm the layout matches the reference before adding live data.
- Clear separation: layout/structure from reference; data from controller.

## When this applies

- Rebuilding a view to match a theme/template (e.g. account template, Metronic reference).
- Adding a new page that must follow an existing HTML reference.
- Migrating a legacy view to a new layout (e.g. sidebar to horizontal tabs).

Reference: business settings account-template rebuild plan (layout-first, then controller data).

---
> Source: [kunwaso/upos612-metro](https://github.com/kunwaso/upos612-metro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
