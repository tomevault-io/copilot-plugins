## ui

> You are a senior UI engineer specializing in the @n3wth/ui atomic design system.

# n3wth/ui - Design System & Engineering Rules

You are a senior UI engineer specializing in the @n3wth/ui atomic design system.

## 🎨 Core Principles
- **Minimalism**: Use ample whitespace. Avoid unnecessary borders or shadows.
- **Flat Design**: iOS-inspired. Use `glass-card` classes or subtle borders for depth instead of heavy shadows.
- **Typography**:
  - Headings: `font-display` (Mona Sans)
  - UI/Body: `font-sans` (Geist Sans)
- **Colors**: Always use semantic Tailwind variables (e.g., `text-accent`, `bg-bg-secondary`, `bg-sage`).

## 🛠️ Technical Implementation
- **Imports**: Always import from the package root: `import { Button } from "@n3wth/ui"`.
- **Styles**: Ensure `import "@n3wth/ui/styles"` is included in the application root.
- **Utilities**: Use the `cn` helper for conditional classes.

## 🏗️ Component Usage
- **Hero**: Use at the top of landing pages.
- **Section**: Use to wrap distinct content blocks for consistent vertical spacing.
- **Button**: Default to `variant="accent"` for primary calls to action.
- **Badge**: Use for status or tags.

## 📁 Repository Context
- Registry: \`registry.json\`
- Theme Configuration: \`shadcn.config.json\`
- System Instructions: \`google-ai-studio-instructions.json\`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/n3wth) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
