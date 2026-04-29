## marketing-demos

> For every landing page route, co‑locate a route‑specific component library that re‑styles generic shadcn UI primitives.

# Per‑Route Component Libraries

For every landing page route, co‑locate a route‑specific component library that re‑styles generic shadcn UI primitives.

## Structure

- Base shadcn primitives live in `[src/components/ui](mdc:src/components/ui)`.
- Each route has its own library at `[src/app/<route>/ui](mdc:src/app)` that composes the base primitives and applies local styles/tokens defined by that route’s theme, e.g. `[src/app/v2/theme.module.css](mdc:src/app/v2/theme.module.css)`.
- Route code (pages, sections, components) MUST import from its local `ui` folder and never from the base directly.

Example for route `v2`:

```
src/app/v2/
  layout.tsx           # installs route theme variables
  theme.module.css     # route palette/tokens
  page.tsx
  ui/
    button.tsx         # wraps base Button with v2 styles
    input.tsx          # wraps base Input with v2 styles
  components/...
```

## Implementation rules

- Local `ui/*` components should import from the base and extend styles:

```tsx
// src/app/v2/ui/button.tsx
import { Button as BaseButton } from "@/components/ui/button";

export function Button(props: React.ComponentProps<typeof BaseButton>) {
  return <BaseButton className="rounded-[14px] bg-black text-white" {...props} />;
}
```

- Route THEME is owned by the route: set CSS variables in the route `layout.tsx` using `[theme.module.css](mdc:src/app/v2/theme.module.css)`; local `ui` components should reference those variables.
- If a needed primitive does not exist in the route `ui/`, create it by composing the base first; do not import the base directly in route pages/sections.

## References in this repo

- Variant containers: `[src/app/v1/layout.tsx](mdc:src/app/v1/layout.tsx)`, `[src/app/v2/layout.tsx](mdc:src/app/v2/layout.tsx)`, `[src/app/v3/layout.tsx](mdc:src/app/v3/layout.tsx)`
- Themes: `[src/app/v1/theme.module.css](mdc:src/app/v1/theme.module.css)`, `[src/app/v2/theme.module.css](mdc:src/app/v2/theme.module.css)`, `[src/app/v3/theme.module.css](mdc:src/app/v3/theme.module.css)`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/formalco) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
