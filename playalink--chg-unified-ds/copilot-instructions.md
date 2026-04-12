## chg-unified-ds

> **IMPORTANT:** This project uses custom token scales from Figma, NOT Tailwind defaults.


# Tailwind Token Scales

**IMPORTANT:** This project uses custom token scales from Figma, NOT Tailwind defaults.

## Spacing Scale

Pixel values, not Tailwind's 4x multiplier:
- `px-8` = 8px, `py-4` = 4px, `gap-6` = 6px
- Available: 0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 24, 32, 48, 64, 80

## Border Radius Scale

- `rounded-6` = 6px, `rounded-8` = 8px
- Available: 0, 2, 4, 6, 8, 12, 16, 20, 24, 9999
- Do NOT use `rounded-md`, `rounded-lg` - these don't exist

## Colors

- Use `text-base-white` and `bg-base-white` (not `text-white`, `bg-white`)
- Brand colors: `brand-50` through `brand-950`
- Semantic: `error-*`, `warning-*`, `success-*`
- Gray scale: `gray-50` through `gray-950`

## Examples

```tsx
// Correct
<div className="px-8 py-4 rounded-6 bg-brand-600 text-base-white">

// Wrong - these classes don't exist
<div className="px-2 py-1 rounded-md bg-white text-white">
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PlayaLink) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
