## a2ui-bridge

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A2UI Bridge implements Google's [A2UI Protocol](https://a2ui.org) for React, enabling AI-generated UIs that render with any component library. The protocol uses declarative JSON (not executable code) to describe UI structures, which adapters then render using the target design system.

## Development Commands

```bash
# Install dependencies (uses pnpm workspaces)
pnpm install

# Build all packages (Turborepo handles dependency order)
pnpm run build

# Run demo application
cd apps/demo && pnpm run dev

# Build and lint in parallel
pnpm run build
pnpm run lint

# Clean all build artifacts
pnpm run clean

# Run a single package's build
pnpm --filter @a2ui-bridge/core build
pnpm --filter @a2ui-bridge/react build
pnpm --filter @a2ui-bridge/react-shadcn build
```

## Architecture

### Package Dependency Chain

```
@a2ui-bridge/core (protocol processing, framework-agnostic)
        ↓
@a2ui-bridge/react (React bindings, hooks, Surface component)
        ↓
@a2ui-bridge/react-shadcn (ShadCN/Tailwind adapters)
        ↓
apps/demo (interactive demo with multi-provider LLM support)
```

### Core Data Flow

1. **AI generates A2UI JSON messages** (beginRendering, surfaceUpdate, dataModelUpdate)
2. **A2uiMessageProcessor** parses messages and builds component tree from raw ComponentInstance data
3. **Surface component** renders tree using registered component adapters
4. **Adapters** translate A2UI node properties to target library props

### Key Abstractions

**A2uiMessageProcessor** (`packages/core/src/processor.ts`):
- Manages multiple surfaces (each surface is an independent UI tree)
- Handles data model binding (paths like `/form/email` resolved to values)
- Rebuilds component trees incrementally on each surfaceUpdate

**Adapter Pattern** (`packages/react/src/adapter.ts`):
- `createAdapter(Component, { mapProps })` - wraps any React component for A2UI
- `extractValue()` - unwraps A2UI DataValue types (literalString, literalNumber, path bindings)
- `createActionHandler()` - generates onClick handlers that dispatch UserAction events

**Component Types** (from `packages/core/src/types.ts`):
- Layout: Row, Column, Card, List, Grid, Tabs
- Input: TextField, Checkbox, Select, Slider, DateTimeInput
- Display: Text, Image, Icon, Badge, Divider
- Interactive: Button, Modal

### API Serverless Functions

The `/api/` directory contains Vercel serverless functions that proxy LLM requests:
- `generate.js` - Multi-provider (Anthropic, OpenAI, Google) message generation
- `providers.js` - Lists available LLM providers

These use Node.js runtime with 60s timeout for MCP Enhanced mode.

### MCP Server

`apps/demo/mcp-server/` provides an MCP server for Claude Desktop that exposes:
- `list_components` - Available A2UI components by category
- `get_component_schema` - Detailed props and examples for a component
- `get_layout_patterns` - Pre-built patterns (login, dashboard, wizard, etc.)
- `validate_a2ui` - Validates A2UI JSON against schema

## Adding New Components

1. **Define in core** (`packages/core/src/components.ts`): Add interface with required/optional props
2. **Add guard** (`packages/core/src/guards.ts`): Type guard function
3. **Add node type** (`packages/core/src/types.ts`): Add to AnyComponentNode union
4. **Handle in processor** (`packages/core/src/processor.ts`): Add switch case in buildNodeRecursive
5. **Create adapter** (`packages/react-shadcn/src/components/`): Map A2UI props to ShadCN component
6. **Register** (`packages/react-shadcn/src/mapping.ts`): Add to shadcnComponents object

## A2UI Protocol Concepts

**StringValue types**: Properties often use `{ literalString: "value" }` or `{ path: "/data/key" }` - use `extractValue()` to unwrap

**Children references**: Use `ComponentArrayReference` with `explicitList: ["id1", "id2"]` or `template: { componentId, dataBinding }`

**Data binding**: Components can bind to data model paths; processor resolves these during tree building

**Surface lifecycle**: beginRendering sets root → surfaceUpdates add/modify components → dataModelUpdate updates bound values

---
> Source: [southleft/a2ui-bridge](https://github.com/southleft/a2ui-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
