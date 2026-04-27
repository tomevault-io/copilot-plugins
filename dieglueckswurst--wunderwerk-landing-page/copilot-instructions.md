## wunderwerk-landing-page

> Color system conventions – ensures all colors use centralized CSS variables and Tailwind classes


# Color System

All colors in this project are defined as CSS custom properties in `src/index.css` (`:root` block) and mapped to Tailwind utility classes via `tailwind.config.ts`.

## Rules

1. **Never use hardcoded color values** (hex, rgb, hsl literals) in components or CSS files.
2. **Always use Tailwind classes** that reference the design tokens: `bg-warm`, `text-warm`, `bg-background`, `text-foreground`, `border-warm/30`, etc.
3. **To add a new color**, follow this order:
   - Add CSS variable in `src/index.css` under `:root` (and `.dark` if needed)
   - Add Tailwind mapping in `tailwind.config.ts` → `theme.extend.colors`
   - Use the Tailwind class in components

## Key Tokens

- `--background` / `bg-background` – page background (warm off-white)
- `--warm` / `bg-warm`, `text-warm` – primary brand color (pastel rust-red)
- `--warm-hover` / `bg-warm-hover` – hover state for warm
- `--warm-strong` / `bg-warm-strong` – stronger variant of warm
- `--foreground` / `text-foreground` – default text color
- `--primary` / `bg-primary` – primary UI color (dark)
- `--input-bg` / `bg-input-bg` – input/textarea background (white, distinct from page background)

## Examples

```tsx
// BAD – hardcoded hex
<div style={{ backgroundColor: '#f7f5f1' }}>
<Button className="bg-[#c4705a] hover:bg-[#b5634f]">

// GOOD – design tokens
<div className="bg-background">
<Button className="bg-warm hover:bg-warm-hover text-white">
```

```css
/* BAD */
background-color: #f7f5f1;

/* GOOD */
background-color: hsl(var(--background));
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DieGlueckswurst) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
