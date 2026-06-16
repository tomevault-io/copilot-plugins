## ui-kit

> **Monorepo Structure**: Turborepo with pnpm workspaces

# SNAP-UI Project Architecture

## Core Architecture

**Monorepo Structure**: Turborepo with pnpm workspaces

- `packages/ui-kit` (@snapverse/ui-kit): Component library (React 19, TypeScript)
- `packages/stylish` (@snapverse/stylish): Styling system (empty/WIP)
- `apps/docs`: Next.js 15 demo app (port 3001)
- `apps/web`: Next.js 15 app (port 3000)

## Critical Patterns

### Component Architecture

- **Namespace Pattern**: Components use compound pattern (e.g., `Accordion.Root`, `Accordion.Item`)
- **Export Structure**: `src/{component}/index.ts` → `namespace.ts` → `{component}.tsx`
- Each component folder exports via namespace pattern for cleaner API

### Type System

- Global types in `packages/ui/types/index.d.ts` under `SnapUI` namespace
- Import types via path alias: `import SnapUI from "@types"`
- `ComponentProps` extends HTMLDivElement attributes + React.ReactNode
- `Merge<Left, Right>` utility type prioritizes right side over left

### Component Pattern Example

```tsx
// {component}.tsx
export const ComponentRoot = forwardRef<HTMLDivElement, Props>(...)
export const ComponentItem = forwardRef<HTMLDivElement, Props>(...)

// namespace.ts
import { ComponentRoot as Root, ComponentItem as Item } from "./{component}"
const Component = { Root, Item }
export default Component
```

### Package Exports

UI package uses wildcard exports: `./*` maps to `./src/*/index.ts`

- Consumers import: `import Accordion from "@snapverse/ui-kit/accordion"`

### Code Generation

Turbo generator at `packages/ui/turbo/generators/config.ts`:

- Run: `pnpm --filter @snapverse/ui-kit generate:component`
- Uses Handlebars template with kebabCase/pascalCase helpers
- Auto-appends export to package.json

### Build System

- Turbo tasks: build depends on `^build` (topological)
- UI package compiles TS → `dist/` (no bundler, just tsc)
- Next.js apps use Turbopack in dev mode
- `dev` task is persistent, non-cacheable

### TypeScript Config

- Base: NodeNext modules, strict mode, ES2022
- React libs extend base + add `jsx: "react-jsx"`
- Path alias `@types` → `./types/index.d.ts`

### Display Names

All components use `@snapverse/{ComponentName}` for React DevTools

## Key Constraints

- Node.js >=22 required
- React 19.1.0 (latest)
- pnpm@9.0.0 package manager
- All workspace packages use `workspace:*` protocol
- ESLint max warnings = 0 (strict)
- Components are forwardRef wrappers using createElement (not JSX)

## Development Preferences

**CRITICAL**:

- **Native-first**: Prefer vanilla JS/TS, native Web APIs, and built-in React features over external libraries
- Only add dependencies when absolutely necessary or explicitly requested
- **Never run builds**: Only modify code when requested. Building is the user's responsibility
- Don't suggest `pnpm build` or `npm build` unless explicitly asked

## Development Flow

1. Generate component: `pnpm --filter @snapverse/ui-kit generate:component`
2. Edit in `packages/ui/src/{component}/`
3. Export via namespace pattern in `namespace.ts`
4. Apps auto-reload via turborepo watch mode
5. Type checking: `pnpm check-types`

## Empty/Placeholder Packages

- `packages/stylish`: Has structure but empty implementation files
- `packages/ui/src/box`: Empty index.ts

---
> Source: [snapverse/ui-kit](https://github.com/snapverse/ui-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
