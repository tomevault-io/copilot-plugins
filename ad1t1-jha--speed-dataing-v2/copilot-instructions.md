## speed-dataing-v2

> - **Framework**: shadcn/ui (New York style)

# Cursor Rules - Design System & File Organization

## Design System

### UI Component Library

- **Framework**: shadcn/ui (New York style)
- **Styling**: Tailwind CSS v4 with CSS variables
- **Color System**: OKLCH color space with CSS variables for theming
- **Component Variants**: Use `class-variance-authority` (cva) for component variants
- **UI Primitives**: Radix UI primitives for accessible components
- **Icons**: Lucide React (`lucide-react`)
- **Animations**: Framer Motion (`framer-motion`)
- **Utility Function**: Use `cn()` from `@/lib/utils` for className merging (combines clsx + tailwind-merge)

### Design Tokens

- Colors are defined in `src/app/globals.css` using CSS variables
- Supports dark mode via `.dark` class
- Border radius: `--radius` (0.625rem default) with variants (sm, md, lg, xl)
- Use semantic color tokens: `primary`, `secondary`, `muted`, `accent`, `destructive`, etc.
- Always use CSS variable-based colors, never hardcoded color values

### Component Patterns

- All UI components should accept `className` prop and merge with `cn()`
- Use `data-slot` attributes for component identification
- Components should be composable and follow shadcn/ui patterns
- Use TypeScript for all components with proper type definitions
- Export component variants when using `cva` for external customization

## File Organization

### Directory Structure

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # API routes
│   ├── layout.tsx         # Root layout
│   ├── page.tsx           # Pages
│   └── globals.css        # Global styles and design tokens
│
├── components/            # React components
│   ├── ui/               # shadcn/ui components (reusable, generic)
│   └── [feature]/        # Feature-specific components (e.g., animated-box.tsx)
│
├── lib/                   # Utility functions and helpers
│   ├── utils.ts          # Core utilities (cn function)
│   ├── utils/            # Additional utility modules
│   └── api/              # API-related utilities and clients
│
├── hooks/                 # Custom React hooks
├── contexts/              # React contexts
├── providers/             # React providers (theme, auth, etc.)
├── types/                 # TypeScript type definitions
│
└── server/                # Server-side code (Next.js server components/actions)
    ├── config/           # Server configuration
    ├── repositories/     # Data access layer
    └── services/         # Business logic services
```

### File Naming Conventions

- **Components**: PascalCase (e.g., `Button.tsx`, `AnimatedBox.tsx`)
- **Hooks**: camelCase with `use` prefix (e.g., `useAuth.ts`, `useTheme.ts`)
- **Utilities**: camelCase (e.g., `utils.ts`, `apiClient.ts`)
- **Types**: PascalCase (e.g., `User.ts`, `ApiResponse.ts`)
- **Server files**: camelCase (e.g., `userService.ts`, `authRepository.ts`)

### Path Aliases

Use these TypeScript path aliases (configured in `tsconfig.json`):

- `@/components` → `src/components`
- `@/components/ui` → `src/components/ui`
- `@/lib` → `src/lib`
- `@/lib/utils` → `src/lib/utils`
- `@/hooks` → `src/hooks`

Always use path aliases instead of relative imports when importing from these directories.

### Component Placement Rules

1. **UI Components** (`src/components/ui/`):

   - Generic, reusable components from shadcn/ui
   - Components that are part of the design system
   - Examples: Button, Card, Input, Dialog, etc.
   - Should be framework-agnostic and highly reusable

2. **Feature Components** (`src/components/`):

   - Feature-specific or business logic components
   - Components that compose UI components
   - Examples: `AnimatedBox`, `UserProfile`, `Dashboard`, etc.
   - Can import from `@/components/ui`

3. **Server Code** (`src/server/`):

   - Server-side logic, data fetching, business logic
   - Use for Next.js Server Components, Server Actions, API route handlers
   - Separate concerns: config, repositories (data access), services (business logic)

4. **Hooks** (`src/hooks/`):

   - Custom React hooks for shared logic
   - Should be reusable across multiple components
   - Examples: `useLocalStorage`, `useDebounce`, `useAuth`

5. **Contexts** (`src/contexts/`):

   - React Context providers for global state
   - Examples: `ThemeContext`, `AuthContext`

6. **Providers** (`src/providers/`):

   - React component providers that wrap the app
   - Examples: `ThemeProvider`, `AuthProvider`

7. **Types** (`src/types/`):
   - Shared TypeScript type definitions
   - Interfaces, types, enums used across the application

### Import Organization

When importing, follow this order:

1. React and Next.js imports
2. Third-party library imports
3. Internal imports (using path aliases)
4. Type imports (use `import type` when possible)

Example:

```typescript
import { useState } from "react";
import { motion } from "framer-motion";
import { Button } from "@/components/ui/button";
import { cn } from "@/lib/utils";
import type { User } from "@/types";
```

### Component Structure Template

```typescript
import * as React from "react";
import { cn } from "@/lib/utils";
// ... other imports

interface ComponentProps extends React.ComponentProps<"div"> {
  // Component-specific props
}

export function Component({ className, ...props }: ComponentProps) {
  return (
    <div
      data-slot="component"
      className={cn("base-classes", className)}
      {...props}
    />
  );
}
```

## Best Practices

1. **Always use `cn()` utility** for className merging
2. **Use CSS variables** for colors, spacing, and design tokens
3. **Support dark mode** by using semantic color tokens
4. **Export component variants** when using `cva` for flexibility
5. **Use path aliases** (`@/`) instead of relative imports
6. **Type everything** - use TypeScript for all files
7. **Follow Next.js conventions** - use App Router patterns
8. **Separate concerns** - keep UI, logic, and data access separate
9. **Compose components** - build complex components from simpler UI components
10. **Use `data-slot` attributes** for component identification and styling

## Adding New Components

- **UI Components**: Add to `src/components/ui/` and follow shadcn/ui patterns
- **Feature Components**: Add to `src/components/` at root level or in feature folders
- **Use the `ui:add` script**: Run `pnpm ui:add [component-name]` to add shadcn/ui components
- **Always export** components and their types/variants when applicable

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ad1t1-jha) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
