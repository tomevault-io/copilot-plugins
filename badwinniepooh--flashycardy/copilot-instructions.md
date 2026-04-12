## flashycardy

> **ALWAYS use shadcn/ui components for UI design.** It is strictly **NOT ALLOWED** to create custom UI elements if a shadcn component exists for the same usage.

# shadcn/ui Component Usage

## Core Requirement

**ALWAYS use shadcn/ui components for UI design.** It is strictly **NOT ALLOWED** to create custom UI elements if a shadcn component exists for the same usage.

## Guidelines

1. **Check shadcn/ui First**: Before creating any UI component, always check if shadcn/ui has a component for that purpose at https://ui.shadcn.com/docs/components
2. **Use Existing Components**: If a shadcn component exists, you MUST use it instead of building a custom alternative
3. **Installation**: Install shadcn components using: `npx shadcn@latest add [component-name]`
4. **Customization**: If needed, customize shadcn components using Tailwind CSS classes rather than creating new components from scratch
5. **Component Library**: Common shadcn components include:
   - Buttons
   - Cards
   - Dialogs/Modals
   - Forms (Input, Select, Checkbox, Radio, etc.)
   - Navigation (Tabs, Menus, Dropdowns)
   - Feedback (Toast, Alert, Progress)
   - Data Display (Table, Badge, Avatar)
   - And many more

## What to Do

✅ Use shadcn/ui components whenever available
✅ Combine shadcn components to build complex UIs
✅ Customize shadcn components with Tailwind CSS
✅ Follow shadcn's composition patterns

## What NOT to Do

❌ Create custom button components when shadcn Button exists
❌ Build custom modals when shadcn Dialog exists
❌ Implement custom form inputs when shadcn Form components exist
❌ Reinvent any UI element that shadcn already provides

## Exception

Only create custom UI components when shadcn/ui does NOT provide a component for that specific use case.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BadWinniePooh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BadWinniePooh)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
