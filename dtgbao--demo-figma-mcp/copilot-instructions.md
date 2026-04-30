## demo-figma-mcp

> Nuxt UI theming for Figma implementation—prefer global design system (app.config, CSS tokens) over per-component styling

# Nuxt UI: Figma → code and component work

When translating Figma or building UI with **Nuxt UI**, change the **design system first**, then compose components. Avoid scattering one-off `class` / `ui` overrides when the same pattern appears more than once.

## Priority order (highest first)

1. **`app/app.config.ts`** — Runtime theme: map semantic colors (`ui.colors`) and **global** component themes (`ui.button`, `ui.card`, …) using the same shape as each component’s theme (slots, variants, compoundVariants, defaultVariants).
2. **`app/assets/css/main.css`** — Foundation tokens: `@theme` / `@theme static` for fonts, palette steps (`--color-*-50` … `950`), breakpoints. Override Nuxt UI surfaces via `:root` / `.dark` (`--ui-bg`, `--ui-primary`, `--ui-container`, etc.) when specs demand it.
3. **`nuxt.config.ts`** — Register **extra** semantic color names in `ui.theme.colors` before using them in `app.config.ts` `ui.colors`.
4. **Component level** — Use `ui` prop, `class`, or inline utilities only for **genuinely local** exceptions (single screen, prototype, or true one-off).

## Semantic colors

Use Nuxt UI semantics (`primary`, `secondary`, `success`, `info`, `warning`, `error`, `neutral`) in props and classes (`text-primary`, `bg-muted`). Map Figma “brand / accent / danger” to these names in `app.config.ts`, not arbitrary hex on each node.

## Figma mapping habits

- **8px grid for spacing** — Treat layout spacing as multiples of **8px** (8pt grid). Map Figma distances to the nearest grid step and express them with **scale utilities** (`p-2` / `p-4` / `p-6` / `p-8`, `gap-*`, `space-*`, `m-*`, etc.), not one-off pixels.
- **No arbitrary spacing values** — Do **not** use bracket arbitrary spacing like `p-[13px]`, `m-[13px]`, `gap-[19px]`, `mt-[7px]`. If the design needs a recurring non-standard step, add a **named token** in `@theme` (e.g. `--spacing-*`) and use the matching utility; otherwise round to the closest standard step.
- **Canonical CSS variable utility syntax** — When using CSS variables in utility classes, prefer Tailwind canonical shorthand `utility-(--token)` instead of `utility-[var(--token)]` (e.g. `rounded-(--agentic-radius-lg)`, `bg-(--agentic-gradient-primary)`, `shadow-(--agentic-shadow-card)`).
- Repeated radii, spacing, button styles, card chrome → **defaultVariants** and **slots** in `app.config.ts` under `ui.<component>`, not copy-pasted classes per file.
- New brand palettes → define full scales in `@theme static`, then point `ui.colors` at the Tailwind color name.
- Light/dark or surface hierarchy → prefer **CSS variables** documented for Nuxt UI (`--ui-*`) over hard-coding per component.

## When component-level styling is OK

- Layout-only wrappers (spacing between sections, grid columns).
- Content that has no matching theme hook.
- A/B or temporary experiments—then promote winners into `app.config.ts` or `main.css`.

```ts
// Prefer: global defaults in app.config.ts
export default defineAppConfig({
  ui: {
    colors: { primary: 'green', neutral: 'slate' },
    button: {
      defaultVariants: { color: 'primary', size: 'md' },
      slots: { base: 'font-semibold' }
    }
  }
})
```

```vue
<!-- Avoid as default pattern: pushing design system into every instance -->
<UButton class="font-semibold rounded-xl" color="primary" />
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dtgbao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
