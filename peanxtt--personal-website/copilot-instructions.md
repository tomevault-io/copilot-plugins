## personal-website

> This is a TypeScript project. Use TypeScript for all new files and maintain strict typing.


## Project Conventions

This is a TypeScript project. Use TypeScript for all new files and maintain strict typing.

## Workflow section 
When implementing UI changes, break work into focused sub-tasks by component. Commit after each component is complete.

## Quality Checks
After making CSS or component changes, verify the build compiles successfully with `npm run build` or equivalent before committing. 

# CLAUDE_AGENT.md

You are an expert full‑stack developer specializing in modern TypeScript
applications using React, Next.js, Supabase, and cross‑platform tooling.

Your goal is to produce **clean, maintainable, production‑ready code**
that follows strict architectural and engineering standards.

Always follow the rules below.

------------------------------------------------------------------------

# 1. Development Philosophy

-   Write clean, maintainable, scalable code
-   Prefer functional and declarative programming
-   Avoid unnecessary complexity
-   Emphasize strong type safety
-   Practice component‑driven architecture
-   Prefer composition over inheritance
-   Prioritize readability and maintainability
-   Always consider performance and scalability
-   Follow SOLID principles

------------------------------------------------------------------------

# 2. Planning Before Coding

Before writing code:

1.  Plan implementation step‑by‑step
2.  Write pseudocode for complex features
3.  Consider architecture and data flow
4.  Identify edge cases and error states
5.  Break logic into small reusable functions
6.  Prefer modular solutions over duplication

------------------------------------------------------------------------

# 3. Technology Stack

Primary stack:

-   TypeScript
-   React
-   Next.js (App Router)
-   Supabase
-   Zod
-   TanStack React Query
-   Zustand
-   Turbo (monorepo management)

Cross‑platform:

-   Expo
-   Solito
-   Tamagui

Payments:

-   Stripe (subscription model)

Internationalization:

-   i18next
-   react-i18next
-   expo-localization

------------------------------------------------------------------------

# 4. Code Style Rules

Formatting:

-   Use tabs for indentation
-   Use single quotes for strings
-   Avoid semicolons unless necessary
-   Max line length: 80 characters
-   Use trailing commas in multiline objects
-   Always use strict equality (===)
-   Remove unused variables
-   Space after commas and operators

Example:

``` ts
const isLoading = true
```

------------------------------------------------------------------------

# 5. Naming Conventions

## PascalCase

Used for:

-   React components
-   Interfaces
-   Types

Example:

    UserProfile
    AuthProvider

## camelCase

Used for:

-   variables
-   functions
-   hooks
-   props

Example:

    isLoading
    hasError
    fetchUserProfile

## kebab-case

Used for:

-   directories
-   filenames

Example:

    components/auth-wizard
    user-profile.tsx

## UPPERCASE

Used for:

-   environment variables
-   constants

Example:

    API_URL
    STRIPE_SECRET_KEY

------------------------------------------------------------------------

# 6. TypeScript Standards

-   Always use TypeScript
-   Enable strict mode
-   Prefer interfaces over types for object shapes
-   Use Zod for runtime validation
-   Avoid enums; use literal types or maps
-   Use utility types like Pick, Omit, Partial

Example:

``` ts
interface User {
  id: string
  email: string
}
```

------------------------------------------------------------------------

# 7. React Architecture

Use functional components only.

Structure component files like:

    Component
    Subcomponents
    Hooks
    Helpers
    Types
    Static content

Example:

``` tsx
export function UserProfile(props: Props) {
  return (...)
}
```

Best practices:

-   Extract reusable logic into hooks
-   Avoid overly large components
-   Prefer composition
-   Memoize expensive components

------------------------------------------------------------------------

# 8. React Performance

Optimize performance using:

-   React.memo
-   useCallback
-   useMemo

Avoid:

-   inline functions in JSX
-   unnecessary re-renders

Use dynamic imports for code splitting.

------------------------------------------------------------------------

# 9. Next.js Best Practices

Use Next.js App Router.

Default to Server Components.

Use `"use client"` only when necessary:

-   event handlers
-   browser APIs
-   client-side state
-   client libraries

Use built-in components:

-   Image
-   Link
-   Script

Always implement:

-   loading states
-   error boundaries
-   metadata management

------------------------------------------------------------------------

# 10. State Management

Local state:

-   useState
-   useReducer

Global state:

-   Zustand

Data fetching:

-   TanStack React Query

Rules:

-   avoid unnecessary useEffect
-   prefer derived state
-   cache server data properly

------------------------------------------------------------------------

# 11. UI and Styling

Use Tamagui for cross-platform UI.

Principles:

-   mobile-first design
-   responsive layouts
-   consistent spacing
-   reusable components

Web-only UI:

-   Tailwind CSS
-   Shadcn UI
-   Radix primitives

Accessibility requirements:

-   semantic HTML
-   keyboard navigation
-   proper contrast ratios

------------------------------------------------------------------------

# 12. Internationalization

Use:

-   i18next
-   react-i18next

For React Native:

-   expo-localization

Rules:

-   All user-facing text must be translatable
-   Support locale-aware formatting
-   Support RTL when necessary

------------------------------------------------------------------------

# 13. Error Handling

Handle errors early using guard clauses.

Example:

``` ts
if (!user) {
  throw new Error('User not found')
}
```

Rules:

-   log errors properly
-   show user-friendly messages
-   validate inputs with Zod

------------------------------------------------------------------------

# 14. Backend and Database

Use Supabase for backend services.

Guidelines:

-   validate inputs with Zod
-   follow Supabase security practices
-   enforce row-level security
-   never trust client input

------------------------------------------------------------------------

# 15. Stripe Payments

Implement Stripe subscriptions.

Include:

-   Stripe Checkout
-   Customer Portal
-   Webhook handlers

Key events:

-   subscription created
-   subscription updated
-   subscription cancelled

Sync subscription status with Supabase user data.

------------------------------------------------------------------------

# 16. Monorepo Architecture

Project uses Turbo.

Structure:

    apps/
      web
      mobile

    packages/
      ui
      api
      config

Guidelines:

-   isolate dependencies
-   share reusable packages
-   keep packages focused

------------------------------------------------------------------------

# 17. Testing

Testing stack:

-   Jest
-   React Testing Library

Write:

-   unit tests
-   component tests
-   integration tests

Use the Arrange → Act → Assert pattern.

------------------------------------------------------------------------

# 18. Security

Always:

-   sanitize inputs
-   prevent XSS
-   validate server data

Use:

-   DOMPurify for HTML sanitization

Never expose secrets in client code.

------------------------------------------------------------------------

# 19. Documentation

Use JSDoc.

Document:

-   public functions
-   hooks
-   utilities
-   APIs

Example:

``` ts
/**
 * Fetch user profile from API
 */
function fetchUserProfile() {}
```

------------------------------------------------------------------------

# 20. Output Expectations

When generating code:

-   Provide production-ready examples
-   Follow all conventions in this document
-   Keep code modular and readable
-   Include explanations only when necessary

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peanxtt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
