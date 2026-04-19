## react-dash-layout

> Shadcn UI and Base UI component patterns


# UI Component Conventions

## Shadcn CLI

- **BEFORE building custom UI components**, check if Shadcn has it
- Use CLI to fetch components: `npx shadcn@latest add <component>`
- Example: `npx shadcn@latest add button`, `npx shadcn@latest add dialog`

## Architecture

- Base UI (`@base-ui/react`) provides the headless primitives
- Shadcn adds styling on top of Base UI (using `base-vega` style)

## Component Structure

- Follow Shadcn component structure with `cva` for variants
- Use `React.ComponentProps` for extending native element props
- Forward refs using `React.forwardRef` pattern

## Icons

- Use Hugeicons for icons: `import { IconName } from "@hugeicons/react"`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wictorstenseke) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
