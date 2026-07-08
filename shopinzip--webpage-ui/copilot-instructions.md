## webpage-ui

> This repo owns the reusable storefront UI components used by the editor preview and live storefront renderer. Use it for section components, variant components, shared visual behavior, and exports.

# Areakart Storefront UI Library

This repo owns the reusable storefront UI components used by the editor preview and live storefront renderer. Use it for section components, variant components, shared visual behavior, and exports.

## Always-On Rules

- Components must accept normalized props and blocks instead of depending on editor-only state or schema logic.
- Keep sections reusable by both `areakart-frontend` preview and `Project-Web` rendering.
- Export variants cleanly so resolver and preview layers can map section type plus variant without hacks.
- New sections must work with the existing global style, color, and font logic. Only keep safe fallback styles inside components.
- Support multiple instances of the same section without shared mutable state or assumptions about ids.

## Repo Boundaries

- Do not add editor schema, registry, or normalization logic here.
- Do not add renderer resolver logic here.
- Keep content prop-driven instead of hardcoding storefront data into the component layer.

## Performance Notes

- Keep marquee, ticker, scroll, and animation behavior lightweight.
- Avoid ResizeObserver loops, expensive DOM measuring loops, huge duplicated arrays, and heavy state churn.
- Video components must be safe when media sources are missing and should not crash if controls cannot initialize.

## Reusable Skill

- See `.agents/skills/storefront-builder/SKILL.md` for the standard workflow when adding or modifying storefront builder modules.

---
> Source: [SHOPINZIP/Webpage-UI-](https://github.com/SHOPINZIP/Webpage-UI-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
