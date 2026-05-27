## design-tokens

> Visual values via CSS variables; no hex / rgb literals in style


# Design-token strict (binding, theme/mode safety)

You are editing UI code. All visual values (color, background, border, shadow, padding / margin / gap, font-size, font-weight, border-radius, transition, z-index) **must** be referenced through CSS variables.

## Token layers

- **Layer 1 raw tokens** — `--g-gray-*`, `--g-space-*`, `--g-radius-*`, `--g-shadow-*`, `--g-transition-*`, `--g-z-*`. Defined in `packages/ui-shared/src/styles/global.css`.
- **Layer 2 semantic tokens** — `--color-*`, `--sidebar-*`, `--shadow-*`, `--space-*`, `--font-size-*`, `--radius-*`, `--transition-*`, `--z-*`. Defined in `light.css` / `dark.css` and flip with `data-theme`.

Components consume **Layer 2** by default (semantic intent). Drop to Layer 1 only when semantically neutral.

## Forbidden

- Hex / `rgb(a)` / `hsl(a)` literals in `*.module.css`.
- Hex / `rgb(a)` literals inside `style={{}}` blocks.
- Raw numeric `padding: 8` / `fontSize: 14` / `borderRadius: 4` etc. in `style={{}}`.

## Three escape hatches

1. **Recharts colours** — must import from `packages/ui-shared/src/theme/chartColors.ts` (the single sanctioned hex source).
2. **CSS variable bridges** — `style={{ '--foo': dynamicValue }}` is fine.
3. **Runtime-computed dimensions** — `` style={{ paddingLeft: `${level * 20}px` }} `` is fine.

Anything else needs an explicit user waiver in chat.

## Enforcement

```bash
npm run check:design-tokens           # CI gate
npm run check:design-tokens:hints     # migration suggestions when refactoring legacy code
```

Skipping this rule requires **explicit user approval** in chat.

---
> Source: [AlphaBitCore/nexus-gateway](https://github.com/AlphaBitCore/nexus-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
