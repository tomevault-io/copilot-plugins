## design-pattern

> Unified Design System for Chapadevs - Modern IT Consultancy (Green/White/Black)


# Chapadevs Unified Design System & Workflow

## 🎯 Component Strategy
- **Hybrid Core**: Use Shadcn for complex layout structures (Sidebar, Tabs, Dialog) and local `ui-components` for atomic elements (Button, Card, Badge, PageTitle).
- **Barrel Export**: All components MUST be exported and imported via `@/components/ui-components/index.js`.
- **Standardization**: Never create one-off styles. Use the library components below.

## 🛠️ Workflow: Adding New Shadcn Components
1. **CLI**: Run `npx shadcn@latest add "component-name"`.
2. **Folder**: Create a new folder `frontend/src/components/shadcn-components/shadcn-"component-name"/`.
3. **Move**: Relocate the generated `.jsx` file into this folder.
4. **Reskin (MANDATORY)**:
   - Replace all `rounded-*` classes with `rounded-none`.
   - Map typography to `font-heading` for headers and `font-body` for content.
5. **Export**: Add `export * from "@/components/shadcn-components/shadcn-folder/file"` to the barrel index.

## 🎨 Visual Identity & Style Guide
- **Sharp Look**: `border-radius: 0` (rounded-none) on everything (Buttons, Cards, Inputs, Modals).
- **Typography**:
  - Headings: `font-heading` (Code Bold, monospace, uppercase).
  - Buttons/Nav: `font-button` (Creato Display Medium).
  - Body: `font-body` (Coolvetica).
- **Color Palette**:
  - Primary: `#059669` (bg-primary / text-primary).
  - Surface: `#ffffff` (bg-surface).
  - Border: `#e5e7eb` (border-border).

## 📑 Component Reference (Legacy Styles)
| Component | Variant / Usage |
|-----------|-----------------|
| `Button` | Variants: `primary`, `secondary`, `ghost`, `danger`. Renders as `Link` if `to` prop is present. |
| `Card` | Variants: `default`, `elevated`, `accent` (green bottom border), `ghost`, `outline`. |
| `Badge` | Status: `holding`, `open`, `ready`, `development`, `completed`. Semantic: `success`, `error`, `neutral`. |
| `PageTitle`| H1 with green left border, Code Bold, Uppercase. |
| `Alert` | Conditionally rendered variants: `error` (red), `success` (green), `info` (blue), `warning` (amber). |

## 🛠️ Sidebar Docking Rule
- The `SidebarTrigger` MUST be absolute-positioned inside the `Sidebar` at `right-[-14px] top-20 z-50` to stay docked to the border.

## 🚀 Pre-flight Checklist
- [ ] Component installed via Shadcn CLI and moved to dedicated folder?
- [ ] All `rounded` classes removed?
- [ ] Exported via `@/components/ui-components`?
- [ ] Content wrapped in `max-w-[1200px] mx-auto` or `<Container>`?

## Rules
  *Dont Children component styles and applying them on Parent components
  *New props should be asked before added
  *Priority to Shadcn components styles
  *No more new CSS files for individual components 

---
> Source: [Chapadevs/Chapadevs](https://github.com/Chapadevs/Chapadevs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
