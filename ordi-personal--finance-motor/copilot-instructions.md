## ui-ux-design-guidelines

> This file describes Sure's design system and how views should be styled

Use the rules below when:

- You are writing HTML
- You are writing CSS
- You are writing styles in a JavaScript Stimulus controller

## Rules for AI (mandatory)

The codebase uses TailwindCSS v4.x (the newest version) with a custom design system defined in [sure-design-system.css](mdc:app/assets/tailwind/sure-design-system.css)

- Always start by referencing [sure-design-system.css](mdc:app/assets/tailwind/sure-design-system.css) to see the base primitives, functional tokens, and component tokens we use in the codebase
- Always prefer using the functional "tokens" defined in @sure-design-system.css when possible.
  - Example 1: use `text-primary` rather than `text-white`
  - Example 2: use `bg-container` rather than `bg-white`
  - Example 3: use `border border-primary` rather than `border border-gray-200`
- Never create new styles in [sure-design-system.css](mdc:app/assets/tailwind/sure-design-system.css) or [application.css](mdc:app/assets/tailwind/application.css) without explicitly receiving permission to do so
- Always generate semantic HTML

---
> Source: [Ordi-personal/finance-motor](https://github.com/Ordi-personal/finance-motor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
