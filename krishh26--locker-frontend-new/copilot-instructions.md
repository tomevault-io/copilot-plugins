## theme-colors

> Enforce theme-adaptive colors — never use hardcoded Tailwind colors


# Theme Colors — No Hardcoded Colors

This project uses a CSS variable-based theme system. **NEVER use hardcoded Tailwind color classes** like `bg-blue-500`, `text-green-600`, `from-indigo-100`, etc. Always use theme-adaptive classes.

## Available Theme Colors

| Semantic Name | CSS Variable | Use For |
|---|---|---|
| `primary` | `--primary` | Main brand, info, blue actions |
| `secondary` | `--secondary` | Secondary actions, warnings |
| `accent` | `--accent` | Success, active, completed |
| `destructive` | `--destructive` | Errors, danger, delete |
| `muted` | `--muted` | Disabled, neutral, inactive |
| `success` | `--success` (alias of accent) | Green/completed states |
| `warning` | `--warning` (alias of secondary) | Yellow/pending states |
| `info` | `--info` (alias of primary) | Blue/informational states |

## Correct Usage

```tsx
// Backgrounds with opacity
className="bg-primary/10 border-primary/30"
className="bg-success/10 text-success"
className="bg-destructive/5 border-destructive/15"

// Text colors
className="text-primary"
className="text-warning"
className="text-muted-foreground"

// Status badges
<Badge className="bg-success/10 text-success">Active</Badge>
<Badge className="bg-warning/10 text-warning">Pending</Badge>
<Badge className="bg-destructive/10 text-destructive">Error</Badge>
<Badge className="bg-muted text-muted-foreground">Inactive</Badge>
```

## Forbidden (DO NOT USE)

```tsx
// NEVER hardcode Tailwind colors
className="bg-blue-500"       // use bg-primary
className="text-green-600"    // use text-success or text-accent
className="bg-red-100"        // use bg-destructive/10
className="text-amber-700"    // use text-warning or text-secondary
className="bg-gray-100"       // use bg-muted
className="from-indigo-50"    // use bg-primary/5 (no gradients)
className="text-red-500"      // use text-destructive
```

## Theme Config Files (must stay in sync)

1. `src/app/globals.css` — CSS variables
2. `src/utils/shadcn-ui-theme-presets.ts` — Shadcn preset
3. `src/utils/tweakcn-theme-presets.ts` — Tweakcn preset

---
> Source: [krishh26/locker_frontend_new](https://github.com/krishh26/locker_frontend_new) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
