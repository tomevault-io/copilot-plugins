## ecua-ticket-app

> You are an expert in Solidity, TypeScript, Node.js, Next.js 14 App Router, React, Vite, Viem v2, Wagmi v2, Shadcn UI, Radix UI, TanStack Query and Tailwind Aria.

# Cursor Rules - Solidity, TypeScript, React & TanStack Query

You are an expert in Solidity, TypeScript, Node.js, Next.js 14 App Router, React, Vite, Viem v2, Wagmi v2, Shadcn UI, Radix UI, TanStack Query and Tailwind Aria.

## Key Principles

- Write concise, technical responses with accurate TypeScript examples.
- Use functional, declarative programming. Avoid classes.
- Prefer iteration and modularization over duplication.
- Use descriptive variable names with auxiliary verbs (e.g., isLoading).
- Use lowercase with dashes for directories (e.g., components/auth-wizard).
- Favor named exports for components.
- Use the Receive an Object, Return an Object (RORO) pattern.

## JavaScript/TypeScript

- Use "function" keyword for pure functions. Omit semicolons.
- Use TypeScript for all code. Prefer interfaces over types. Avoid enums, use maps.
- File structure: Exported component, subcomponents, helpers, static content, types.
- Avoid unnecessary curly braces in conditional statements.
- For single-line statements in conditionals, omit curly braces.
- Use concise, one-line syntax for simple conditional statements (e.g., if (condition) doSomething()).

## Error Handling and Validation

- Prioritize error handling and edge cases:
  - Handle errors and edge cases at the beginning of functions.
  - Use early returns for error conditions to avoid deeply nested if statements.
  - Place the happy path last in the function for improved readability.
  - Avoid unnecessary else statements; use if-return pattern instead.
  - Use guard clauses to handle preconditions and invalid states early.
  - Implement proper error logging and user-friendly error messages.
  - Consider using custom error types or error factories for consistent error handling.

## React/Next.js

- Use functional components and TypeScript interfaces.
- Use declarative JSX.
- Use function, not const, for components.
- Use Shadcn UI, Radix, and Tailwind Aria for components and styling.
- Implement responsive design with Tailwind CSS.
- Use mobile-first approach for responsive design.
- Place static content and interfaces at file end.
- Use content variables for static content outside render functions.
- Minimize 'use client', 'useEffect', and 'setState'. Favor RSC.
- Use Zod for form validation.
- Wrap client components in Suspense with fallback.
- Use dynamic loading for non-critical components.
- Optimize images: WebP format, size data, lazy loading.
- Model expected errors as return values: Avoid using try/catch for expected errors in Server Actions. Use useActionState to manage these errors and return them to the client.
- Use error boundaries for unexpected errors: Implement error boundaries using error.tsx and global-error.tsx files to handle unexpected errors and provide a fallback UI.
- Use useActionState with react-hook-form for form validation.
- Code in services/ dir always throw user-friendly errors that TanStack Query can catch and show to the user.
- Use next-safe-action for all server actions:
  - Implement type-safe server actions with proper validation.
  - Utilize the action function from next-safe-action for creating actions.
  - Define input schemas using Zod for robust type checking and validation.
  - Handle errors gracefully and return appropriate responses.
  - Use import type { ActionResponse } from '@/types/actions'
  - Ensure all server actions return the ActionResponse type
  - Implement consistent error handling and success responses using ActionResponse

## TanStack Query

- Use TanStack Query for all data fetching, caching, and synchronization:
  - Implement QueryClient with proper configuration for optimal performance.
  - Use useQuery for data fetching operations.
  - Use useMutation for data modification operations.
  - Leverage useInfiniteQuery for pagination scenarios.
  - Implement proper query key management with consistent naming conventions.
  - Use query invalidation strategically after mutations.
  - Configure staleTime and cacheTime based on data requirements.
  - Implement optimistic updates for better user experience.
  - Use QueryErrorResetBoundary with React error boundaries.
  - Leverage suspense mode when appropriate with React.Suspense.
  - Implement proper loading and error states using query status.
  - Use queryClient.prefetchQuery for SSR scenarios.
  - Structure query keys as arrays with hierarchical naming (e.g., ['users', userId, 'posts']).
  - Implement custom hooks that wrap useQuery/useMutation for reusability.
  - Use onSuccess, onError, and onSettled callbacks judiciously.
  - Configure retry logic based on error types and business requirements.
  - Implement background refetching with appropriate intervals using refetchInterval.

## Key Conventions

1. Rely on Next.js App Router for state changes.
2. Prioritize Web Vitals (LCP, CLS, FID).
3. Minimize 'use client' usage:
   - Prefer server components and Next.js SSR features.
   - Use 'use client' only for Web API access in small components.
   - Avoid using 'use client' for data fetching or state management.
   
Refer to Next.js documentation for Data Fetching, Rendering, and Routing best practices:
- https://nextjs.org/docs

Refer to TanStack Query documentation for advanced patterns and best practices:
- https://tanstack.com/query/latest/docs/framework/react/overview

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vKail)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vKail)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
