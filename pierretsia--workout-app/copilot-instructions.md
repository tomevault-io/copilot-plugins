## workout-app

> Prefer shadcn/ui components over raw HTML elements in React components


# Prefer shadcn/ui Components

Always use existing shadcn/ui primitives from `src/components/ui/` before writing raw HTML elements with custom Tailwind classes.

## Available primitives

Badge, Button, Card, Input, Label, Checkbox, Switch, Tabs, Dialog, Sheet, Drawer, Popover, Tooltip, DropdownMenu, AlertDialog, Collapsible, Toggle, ToggleGroup, Separator, ScrollArea, Avatar, Table, Alert, Form, Textarea, Calendar, Command, Carousel, Sonner.

## How to apply

1. Check `src/components/ui/` for an existing component that matches the intent.
2. Use the component's built-in `variant` prop when a standard variant fits.
3. If you need custom colors/sizing beyond the variants, pass them via `className` — the shadcn components all accept it and merge via `cn()`.
4. Only fall back to raw elements (`<button>`, `<div>`, `<span>`) when no shadcn primitive covers the use case.

```tsx
// ❌ BAD — raw button with hand-rolled pill styles
<button className="inline-flex items-center rounded-full border px-2.5 py-1 text-xs font-medium bg-emerald-500/15 text-emerald-300">
  {label}
</button>

// ✅ GOOD — Badge with variant + className override
<Badge variant="outline" className="bg-emerald-500/15 text-emerald-300 border-emerald-500/25">
  {label}
</Badge>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PierreTsia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
