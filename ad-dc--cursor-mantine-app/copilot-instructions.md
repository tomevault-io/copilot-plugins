## cursor-mantine-app

> Design system component and coding standards for cursor-mantine-app


# 🧩 Next.js + Mantine Design System Rules

## Tech Stack

- **Next.js v14+** App Router
- **Mantine v9** for UI and styling
- **TypeScript**

---

## Styling

- Style only with Mantine system props (`bg`, `c`, `w`, `h`, `p`, `px`, `py`, `m`, `mx`, `my`, `radius`, `shadow`, etc.)
- Do **not** use Tailwind CSS, CSS modules, or inline `style` props

---

## Figma Layout Primitive Mapping

When generating code from Figma selections, automatically detect layout primitives from the generated code's flex/grid properties rather than relying on layer names. See `.cursor/rules/figma-layout-mapping.mdc` for the full decision algorithm and spacing token mapping.

Quick reference:
- `flex-direction: column` → `Stack`
- `flex-direction: row` → `Inline`
- `grid` / `grid-cols-*` → `Grid`
- padding-only container → `Box`
- centered content → `Center`
- If a layer IS named `Stack`, `Grid`, or `Inline`, honor that name as an override

---

## Design System Components

**ALWAYS use DS components** from `@/components/DesignSystem` instead of raw Mantine.

```tsx
// ✅ CORRECT
import { Button, TextInput, Alert } from '@/components/DesignSystem';

// ❌ INCORRECT
import { Button, TextInput, Alert } from '@mantine/core';
```

**Available categories:** Buttons, Inputs, Combobox, Navigation, DataDisplay, Overlays, Typography, Misc, Layout, ComplexComponents

If a Mantine component has no DS wrapper yet, create one:

```tsx
import React, { forwardRef } from 'react';
import { [MantineComponent] as Mantine[Component], [MantineComponent]Props as Mantine[Component]Props } from '@mantine/core';

export interface DS[Component]Props extends Mantine[Component]Props {}

export const [Component] = forwardRef<HTML[Element]Element, DS[Component]Props>(
  ({ ...props }, ref) => <Mantine[Component] ref={ref} {...props} />
);
[Component].displayName = '[Component]';
```

Export through the category index and `components/DesignSystem/index.ts`.

---

## Data Tables

Always use the custom `Table` from `@/components/DesignSystem` — never raw Mantine `Table` or third-party libraries.

## Page Headers

Always use `PageContentHeader` from `@/components/DesignSystem` for page/section headers.

---

## Figma Code Connect — Composition Patterns

### Instance Swap for Container Components

When a Figma component has a content slot (Popover, Modal, Drawer, Card, etc.), use `figma.instance()` to enable inline code composition:

```tsx
props: {
  content: figma.instance('content'),
},
example: (props) => (
  <Popover>
    {props.content}
  </Popover>
),
```

**Critical rule:** The instance swap property must be a **direct child** of the component — not nested inside a frame or group. Nested instance swaps will not resolve inline and will render as empty or dead-end links in Dev Mode.

```
✅ WORKS — direct child:
Popover (component)
  └─ TextArea (instance swap 'content')
  └─ .Popover/Core/pointer

❌ DOES NOT WORK — nested in frame:
Popover (component)
  └─ content (frame)
       └─ TextArea (instance swap)
```

### API Guidance

- `figma.instance('propName')` — For instance swap properties. Renders child code **inline** (or as expandable pill). Use this for content slots.
- `figma.children('layerName')` — Finds children inside a named layer. Renders as a **slot link** (drill-down navigation), not inline. Avoid for content composition.
- `figma.nestedProps('layerName', {...})` — For reading properties from nested component instances (e.g., reading TextInput props from inside a Select). Works well for prop passthrough.

### Composition Requirements

Only code-connected components will render their code when placed in a slot. Non-connected components show as `{/* Code Connect Logic Instance */}` or empty. The `needs-connect` tag in Storybook tracks which components still need Code Connect bindings.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ad-dc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
