## flow

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`@crafter-station/flow` is a zero-dependency tree layout and infinite canvas library for React. It provides generic layout primitives for visualizing hierarchical data structures with animated connections.

## Monorepo Structure

- `packages/flow/` - Core library (`@crafter-station/flow`)
- `apps/playground/` - Interactive testing app (port 3001)
- `apps/www/` - Documentation website (port 3000)

## Commands

```bash
bun install              # Install dependencies
bun run build            # Build all packages
bun run dev              # Development (all apps)
bun run dev:playground   # Start playground only (port 3001)
bun run dev:www          # Start docs only (port 3000)
bun run lint             # Lint all packages
bun run check-types      # Type check all packages
bun run format           # Format with Prettier
```

## Tech Stack

- **Monorepo**: Turborepo
- **Package Manager**: Bun
- **Framework**: Next.js 16 with Turbopack
- **Styling**: Tailwind CSS 4 + shadcn/ui
- **Library Build**: tsup (ESM + CJS with types)

## Core Library Architecture (`packages/flow/src/`)

### Layout Engine

`HierarchyGraph<T>` in `core/hierarchy-graph.ts` - Pure layout calculator that computes node positions and edge paths. Supports vertical/horizontal directions per-node.

### React Components

- `ZoomableCanvas` - SVG pan/zoom container with momentum scrolling
- `HierarchyView<T>` - Main component using render props (`renderNode`, `nodeSize`)
- `EdgePath` - Animated SVG connection lines with preset styles
- `NodeContainer` - foreignObject wrapper for positioning HTML in SVG
- `DotGrid` - Animated background grid pattern

### Type System

```typescript
type HierarchyNode = {
  id: string;
  direction?: "vertical" | "horizontal";
  children?: HierarchyNode[];
  [key: string]: unknown;  // Extend with custom properties
};

type SizeFn<T> = (node: T) => { width: number; height: number };
```

### Usage Pattern

```tsx
<ZoomableCanvas>
  <HierarchyView<MyNode>
    data={tree}
    nodeSize={(node) => ({ width: 150, height: 50 })}
    renderNode={(node) => <MyComponent node={node} />}
  />
</ZoomableCanvas>
```

## Workspace Dependencies

Apps use workspace protocol to reference the flow package:
```json
"@crafter-station/flow": "workspace:*"
```

Turborepo ensures `packages/flow` builds before dependent apps.

---
> Source: [crafter-station/flow](https://github.com/crafter-station/flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
