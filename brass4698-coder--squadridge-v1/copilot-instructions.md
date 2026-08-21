## component-rules

> SquadRidge Component Library Rules

# SquadRidge — components

## Rules

1. **Layout**: Dark navy `#0A0F1E`, Space Grotesk + DM Sans, WCAG AA contrast.
2. **A11y**: Never rely on color alone; pair with text/icons for warnings (e.g. Amber `#F5A623`).
3. **Weight**: Small bundles; avoid decorative animation on low-bandwidth paths.
4. **Buttons**: Teal `#0E9AA7` primary; clear pressed/hover states.

## Typography & spacing

- Fluid type with `clamp()` where headings need to scale.
- Tailwind spacing scale; keep rhythm consistent.

## Patterns

- **Sentiment / tension UI**: Show trend with color + label/icon.
- **Interventions**: Slow down / Pull back — large tap targets, calm wording.
- **Mod tools**: Dense tables/cards; readable on dark backgrounds.

## SVG

- Simple, consistent stroke/fill; no flags or weapon imagery.

## Code

- React function components + hooks; Tailwind for styles; explicit TypeScript props/types.
- Prefer props for variants; avoid one-off style escapes unless necessary.

---
> Source: [brass4698-coder/squadridge-v1](https://github.com/brass4698-coder/squadridge-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
