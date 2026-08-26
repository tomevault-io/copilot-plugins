## component-build

> Always perform full component build & integration steps when updating or creating any child component in the frontend. Follow Shadcn patterns, ensure ui-components are exported via index.js, use Tailwind only (no CSS files), and enforce design system. Applies on every change to children components.


# Full Component Build & Integration (on All Child Component Updates)

**Apply EVERY TIME a child component is updated or created in the frontend.**

## 1. Integrate or Update the Component in the Frontend

Whenever you update or create any child component:

- **Barrel Exports**: Ensure the component is exported from `frontend/src/components/ui-components/index.js` (e.g. `export { default as MyComponent } from './MyComponent/MyComponent'` or `export * from './MyComponent'`).
- **Consumers**: Update any page, parent, or consumer to use the new or updated component from `@/components/ui-components`, replacing any custom markup or obsolete imports.
- **Design System**: Strictly follow `.cursor/rules/design-pattern.mdc` for sharp look, typography, and palette. Use existing ui-components or Shadcn where possible instead of new one-off styles.

## 2. Shadcn & Other Library Patterns (apply on every update)

- **Use Shadcn as the base when available** (e.g. Sidebar, Tabs, Dialog, Card, Avatar, etc.).
- **Pattern Requirements**:
  - **Create with CLI**: `npx shadcn@latest add "component-name"` if adding.
  - **Folder Convention**: Place in `frontend/src/components/shadcn-components/shadcn-<component-name>/`.
  - **Local Utility Import**: Use `import { cn } from "@/utils/shadcn"` within Shadcn components.
  - **Re-export via ui-components**: Make accessible from `@/components/ui-components`:
    - Either with `export * from "@/components/shadcn-components/shadcn-<name>/<file>"` in `frontend/src/components/ui-components/<Name>/index.js`, and then `export * from './<Name>'` in the main barrel.
    - Or simply ensure consumers only import from the barrel.
  - **Reskin Mandatory**: Immediately replace all `rounded-*` with `rounded-none` and update to use `font-heading`/`font-body` as per the design system.

For non-Shadcn library components, mirror these patterns: dedicated folder, export through ui-components, and design system compliance.

## 3. ui-components: Import & Styling Rules for All Updates

- **Imports**: Always import UI components from `@/components/ui-components`. Never use internal component paths in app code—add (or verify) the export in the barrel index.
- **No CSS Files**: Do not create or update `.css` files for individual components. Use Tailwind utility classes only.
- **Styling**: Tailwind controls both layout and tweaks. Use design tokens (e.g. `bg-primary`, `font-heading`, `rounded-none`) per the system spec.

## Checklist (for every child component update)

- [ ] Component is exported via `frontend/src/components/ui-components/index.js` (inc. any folder/subfolder index).
- [ ] If using Shadcn (or other library), all folder, app import, and reskinning/re-export rules followed.
- [ ] No new or changed CSS files; all style changes with Tailwind.
- [ ] All call sites and parent components updated to use the exported component, not old markup.
- [ ] Strict adherence to approved design system (sharp edges, standard palette, typography).

**This rule applies AUTOMATICALLY on every update to any child component in the frontend.**

---
> Source: [Chapadevs/Chapadevs](https://github.com/Chapadevs/Chapadevs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
