## frontend-style-rules

> Shared frontend rules for control-ui, desktop-app/ui, and profile-ui


Apply the shared UI rules in `docs/agents/frontend-style-rules.md` when working inside these paths:

- `control-ui/**`
- `desktop-app/ui/**`
- `profile-ui/**`

Key requirements:

- keep reusable types in sibling `types.ts` files
- keep reusable constants in project constants directories
- prefer aliases over deep relative imports
- use project-global CSS variables for colors and shared tokens
- prefer folder-based components
- build forms from single-purpose primitives such as field wrappers, text inputs, selects, checkboxes, and variant-driven buttons
- in `desktop-app/ui`, use toast notifications for transient success feedback after completed actions; do not place inline green success banners inside cards/forms
- CSS class before inline style, always (except for truly dynamic values)
- no duplicated utilities — check lib/ first, create once if needed
- consistent create/install page shell patterns per project
- in `control-ui`, route sections use `TablePanelHeader` with a persistent icon/title/subtitle/search/refresh/CTA header; initial loading renders in content only, header CTAs are disabled, and only the content pane scrolls
- in `control-ui` and `profile-ui`, sidebar destinations and shareable tab-like sections use canonical App Router paths/subroutes, not `?tab=...`, ad hoc query keys such as `profileTab`, `localStorage`, or component-only state
- stable key props for lists (no array index keys)
- accessibility: focus-visible styles on all interactive elements

---
> Source: [evenfire-ai/evenfire](https://github.com/evenfire-ai/evenfire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
