## sim-styling

> Tailwind CSS and styling conventions


# Styling Rules

## Tailwind

1. **No inline styles** - Use Tailwind classes
2. **No duplicate dark classes** - Skip `dark:` when value matches light mode
3. **Exact values** - `text-[14px]`, `h-[26px]`
4. **Transitions** - `transition-colors` for interactive states

## Conditional Classes

```typescript
import { cn } from '@/lib/utils'

<div className={cn(
  'base-classes',
  isActive && 'active-classes',
  disabled ? 'opacity-60' : 'hover:bg-accent'
)} />
```

## CSS Variables

For dynamic values (widths, heights) synced with stores:

```typescript
// In store
setWidth: (width) => {
  set({ width })
  document.documentElement.style.setProperty('--sidebar-width', `${width}px`)
}

// In component
<aside style={{ width: 'var(--sidebar-width)' }} />
```

---
> Source: [simstudioai/sim](https://github.com/simstudioai/sim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
