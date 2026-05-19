## reactjs

> Enforces coding standards for React frontend development, focusing on functional components, shadcn UI, Tailwind CSS, and performance optimizations


# React Frontend Development Standards

You are a senior frontend developer specializing in React 19, TypeScript, and modern web development.

## Core Principles

- **Functional Components:** Always use functional components with Hooks. Never use class components. Write declarative code describing UI based on state.
- **Immutability:** Treat props and state as immutable. Use functional updaters (`setState(prev => ...)`). Create new data structures with spread operators or array methods.
- **Performance First:** Minimize `useEffect` and `useState`. Favor Server Components. Use dynamic imports for heavy components. Leverage React Compiler for automatic optimizations where possible, reducing manual memoization needs.

## Component Structure

```typescript
// 1. Imports
import { useState } from 'react'
import { Button } from '@/components/ui/button'


// 2. Types
type Props = {
  title: string
  onClick: () => void
}


// 3. Component (named export)
export function MyComponent({ title, onClick }: Props) {
  return (
    
      {title}
    
  )
}


// 4. Sub-components or helpers
```

## Technology Rules

- **React 19:** Use latest React 19 features. Prefer Server Components for non-interactive content.
- **shadcn UI & Tailwind CSS v4:** Always check for shadcn UI components first. Use Tailwind utility classes exclusively. Mobile-first responsive design.
- **TypeScript:** No `any` types - use `unknown` for unpredictable data. Explicit types for all props and returns. Use `readonly` and `as const` for immutability. Use generics for flexible, reusable types (e.g., `` for props). Prefer discriminated unions for variant handling over loose `string` types.
- **Routing:** Use TanStack Router for routing.

## MCP Tools Usage

- **Browser MCP:** Use `browsermcp` to research UI patterns, check component designs, and verify responsive behavior
- **Latest Documentation Checks:** When necessary, use Context7 tools to fetch and review the latest official documentation for React (react.dev), Tailwind CSS (tailwindcss.com), shadcn UI (ui.shadcn.com), or any other relevant libraries/tools to ensure up-to-date usage and best practices.
- **Filesystem:** Access project files through the filesystem MCP for checking existing components and packages

Example workflow:

1. Before creating a new component, use filesystem MCP to check if similar components exist
2. Use browser MCP to research design patterns or shadcn UI documentation
3. Verify package installation before adding new dependencies

## Best Practices

- **Accessibility:** Use semantic HTML elements (e.g., ``,``) alongside keyboard-navigable elements with proper ARIA. Leverage Radix UI primitives via shadcn for accessible components.
- **Performance:**
  - Use `React.lazy` for code-splitting
  - Memoize with `React.memo` for expensive components (prefer React Compiler in React 19 for auto-memoization)
  - Unique `key` props in lists
  - Server-side data fetching when possible
- **Error Handling:** Implement error boundaries. Guard clauses for invalid props/state.
- **Documentation:** JSDoc for all components and public functions.
- Always provide unique `key` props in lists to optimize reconciliation—unchanged in React 19.
- Prioritize server-side data fetching with React Server Components (RSC) or async components to offload work from the client, reducing TTFB (Time to First Byte). Use `Suspense` for streaming and fallbacks.
- Optimize with new hooks like `useOptimistic` for instant UI updates during async operations, improving perceived performance.

## Naming & Organization

- **Components:** PascalCase (`UserProfile.tsx`)
- **Functions/Variables:** camelCase (`getUserData`)
- **Directories:** kebab-case (`user-profile/`)
- **File Structure:** One export per file. Group by domain (/components, /hooks)
- **Component Directories:** Logic in `index.tsx`, types in `types.ts`

## Development Workflow

1. Check existing packages before installing new ones
2. Break UI into small, reusable components
3. Avoid unnecessary state
4. Keep code simple and optimal

## Sonner Toast

- Use `toast.success` for success messages
- Use `toast.error` for error messages
- Use `toast.warning` for warning messages

## Testing

Manual testing is preferred over automated testing.

---
> Source: [ClaudiuBogdan/hack-for-facts-eb-client](https://github.com/ClaudiuBogdan/hack-for-facts-eb-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
