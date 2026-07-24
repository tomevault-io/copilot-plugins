## vayu-ui

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VVayu UI is an AI-optimized React UI component library monorepo. It provides reusable components, hooks, a CLI for scaffolding, and an MCP server for AI tool integration.

## MCP-First Enforcement

When you are writing or reviewing Vayu UI usage code in this repository (components, hooks, docs examples, scaffold snippets, or generated usage code), MCP discovery is mandatory:

1. Run discovery before implementation:
   - `find_component` when slug/name is unknown
   - `get_component_summary` before deciding primitives
2. Validate implementation details before finalizing:
   - `get_component_props`, `get_component_variants`, `get_component_states`, `get_component_events`
   - `get_component_a11y` for focusable, overlay, form, feedback, or keyboard-driven UI
3. Validate anti-patterns and install/import correctness:
   - `get_component_do_not`
   - `get_install_guide`
4. Do not invent props, variants, states, CLI commands, or imports from memory when MCP can confirm them.
5. If a non-Vayu custom implementation is used where a Vayu equivalent exists, include a short justification.

## Commands

```bash
# Build all packages
npm run build

# Start dev servers (docs at localhost:3000)
npm run dev

# Lint all packages
npm run lint

# Run tests
npm run test

# Format code
npm run format

# Generate/refactor a component using AI/ not using anymore
npm run refactor <component-name>
```

**Package-specific commands:**

```bash
# CLI tests
cd packages/cli && npm run test

# Docs dev server with increased memory
cd apps/docs && npm run dev
```

## Architecture

### Monorepo Structure

```
├── apps/docs/           # Documentation site (Next.js + Fumadocs)
├── packages/
│   ├── ui/              # Core components and hooks
│   ├── cli/             # vayu-ui CLI (Oclif)
│   ├── registry/        # Shared metadata for CLI & MCP
│   └── mcp/             # MCP server for AI tools
└── scripts/             # Build utilities
```

### Component Pattern

Components use the **compound component pattern** with namespaced subcomponents:

```tsx
// Example: Button with Icon, Badge, Text subcomponents
export const Button = Object.assign(ButtonRoot, { Icon, Badge, Text });

// Usage
<Button>
  <Button.Icon>...</Button.Icon>
  <Button.Text>Click me</Button.Text>
</Button>;
```

Components:

- Use `"use client"` directive for client-side components
- Extend native HTML attributes (e.g., `ButtonHTMLAttributes`)
- Use `forwardRef` for ref forwarding
- Apply semantic design tokens via Tailwind classes

### Design Tokens (Tailwind v4)

Tokens are defined in [apps/docs/src/app/global.css](apps/docs/src/app/global.css) using `@theme`:

**Base Layers:**

- `canvas`, `canvas-content` - App-level backgrounds and text
- `surface`, `surface-content` - Container backgrounds and text
- `elevated`, `elevated-content` - Modals, popovers, dropdowns

**Semantic Colors:**

- `brand`, `brand-content` - Primary actions, CTAs
- `success`, `warning`, `destructive`, `info` - Status colors
- `muted`, `muted-content` - De-emphasized UI

**Structural:**

- `border`, `field`, `focus` - Borders and focus indicators

**Radius:**

- `rounded-control` - Inputs, buttons
- `rounded-surface` - Cards, containers
- `rounded-overlay` - Modals, popovers

**Shadows:**

- `shadow-control` - Buttons, inputs
- `shadow-surface` - Cards, list items
- `shadow-elevated` - Modals, dropdowns

```tsx
// Example usage
<div className="bg-surface text-surface-content rounded-surface shadow-surface">
  <button className="bg-brand text-brand-content rounded-control">Action</button>
</div>
```

### Registry System

Components and hooks are registered in `packages/registry/src/` with metadata:

```ts
{
  name: "Button",
  slug: "button",
  type: "component",
  category: "inputs",
  targetPath: "src/components/ui",
  fileName: "button.tsx",
  dependencies: ["clsx"],
  registryDependencies: [],
}
```

The CLI (`vayu-ui add <slug>`) uses this registry to copy components and install dependencies.

### Fumadocs Documentation

- **MDX files**: `apps/docs/content/docs/` (file path = URL path)
- **Demo components**: `apps/docs/src/components/demos/<name>-demo.tsx`
- **Frontmatter required**: `title` and `description`

```yaml
---
title: Button
description: A clickable button component.
---
```

## Key Files

| Purpose           | Path                                                                             |
| ----------------- | -------------------------------------------------------------------------------- |
| Component exports | [packages/ui/src/components/ui/index.ts](packages/ui/src/components/ui/index.ts) |
| Hook exports      | [packages/ui/src/hooks/index.ts](packages/ui/src/hooks/index.ts)                 |
| Design tokens     | [apps/docs/src/app/global.css](apps/docs/src/app/global.css)                     |
| Registry data     | [packages/registry/src/components.ts](packages/registry/src/components.ts)       |
| Fumadocs config   | [apps/docs/source.config.ts](apps/docs/source.config.ts)                         |
| Refactor script   | [scripts/refactor-component.ts](scripts/refactor-component.ts)                   |

## Component Guidelines

1. **Accessibility**: Accessible compliance (keyboard nav, focus visible, ARIA)
2. **Styling**: Use design tokens, never hardcode colors or pixel values
3. **Pattern**: Prefer compound components for complex UI
4. **Props**: Extend HTMLAttributes, support `asChild` where applicable

---
> Source: [Rugved1652/vayu-ui](https://github.com/Rugved1652/vayu-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
