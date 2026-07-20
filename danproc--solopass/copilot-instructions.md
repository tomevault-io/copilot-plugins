## spacing-utilities

> Spacing and layout utility class preferences for Tailwind CSS


# Spacing and Layout Guidelines

## Avoid space-* utilities
Avoid using `space-y-*` and `space-x-*` classes for spacing between elements.

## Prefer modern layout with gap
Instead, use modern Flexbox or Grid layouts with `gap-*` utilities:

### Use Flexbox with gap:
- `flex gap-4` - for horizontal layouts
- `flex-col gap-4` - for vertical layouts  
- `flex-row gap-4` - explicit horizontal layouts

### Use Grid when appropriate:
- `grid grid-cols-2 gap-4` - for grid layouts
- `grid gap-x-4 gap-y-2` - for different horizontal/vertical gaps

## Why this is better:
- More semantic and predictable spacing
- Works consistently with modern CSS layout systems
- Easier to maintain and debug
- Better browser support for gap properties
- Avoids margin collapse issues

## Examples:

❌ **Avoid:**
```tsx
<div className="space-y-4">
  <div>Item 1</div>
  <div>Item 2</div>
</div>
```

✅ **Prefer:**
```tsx
<div className="flex flex-col gap-4">
  <div>Item 1</div>
  <div>Item 2</div>
</div>
```

---
> Source: [Danproc/SoloPass](https://github.com/Danproc/SoloPass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
