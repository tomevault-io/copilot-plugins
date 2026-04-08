## onecoach-ui

> > **Meta**: UI component library for OneCoach. React/React Native compatible using Radix UI, Tailwind CSS, and design tokens.

# OneCoach UI - Agent Guide

> **Meta**: UI component library for OneCoach. React/React Native compatible using Radix UI, Tailwind CSS, and design tokens.

## Quick Reference

| Attribute      | Value                    |
| :------------- | :----------------------- |
| **Monorepo**   | `submodules/onecoach-ui` |
| **Styling**    | Tailwind CSS v4          |
| **Components** | Radix UI primitives      |
| **Icons**      | Lucide React             |
| **Animations** | Framer Motion            |

---

## Package Structure

```
submodules/onecoach-ui/
├── packages/
│   ├── ui/                 # @onecoach/ui - Component library
│   ├── hooks/              # @onecoach/hooks - Shared React hooks
│   ├── design-system/      # Design system integration
│   └── lib-design-system/  # @onecoach/lib-design-system - Tokens
├── package.json
└── pnpm-workspace.yaml
```

---

## Key Packages

### `@onecoach/ui` (`packages/ui`)

Main component library:

- Components in `src/components/`
- Entry point: `src/index.ts`
- Radix UI + Tailwind + cva (class-variance-authority)

### `@onecoach/hooks` (`packages/hooks`)

Shared React hooks:

- Form handling
- Feature flags
- UI state management

### `@onecoach/lib-design-system` (`packages/lib-design-system`)

Design tokens & theme:

- Colors, spacing, typography
- Animation utilities
- Theme configuration

---

## Development Guidelines

### Styling

```typescript
// ✅ Use design tokens
className = 'bg-primary text-primary-foreground';

// ❌ Never hardcode values
className = 'bg-[#1a1a1a]';
```

- Always use Tailwind utility classes
- Prefer tokens from `@onecoach/lib-design-system`
- Support dark mode with `dark:` variants

### Creating Components

1. Location: `packages/ui/src/components/`
2. Use **Radix UI** for accessible primitives
3. Use **cva** for variants:

```typescript
import { cva } from 'class-variance-authority';

const buttonVariants = cva('inline-flex items-center justify-center rounded-md', {
  variants: {
    variant: {
      default: 'bg-primary text-primary-foreground',
      outline: 'border border-input bg-background',
    },
    size: {
      default: 'h-10 px-4 py-2',
      sm: 'h-9 px-3',
      lg: 'h-11 px-8',
    },
  },
  defaultVariants: {
    variant: 'default',
    size: 'default',
  },
});
```

---

## Navigation Guide

| Task          | Location                      |
| :------------ | :---------------------------- |
| Add component | `packages/ui/src/components/` |
| Add hook      | `packages/hooks/`             |
| Modify tokens | `packages/lib-design-system/` |
| Check exports | `packages/ui/src/index.ts`    |

---

## Technology Stack

| Layer     | Technology                  |
| :-------- | :-------------------------- |
| Framework | React 19 / React Native     |
| Language  | TypeScript                  |
| Styling   | Tailwind CSS v4             |
| Icons     | Lucide React                |
| Motion    | Framer Motion               |
| Build     | Turborepo / pnpm workspaces |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OneCoach-org)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/OneCoach-org)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
