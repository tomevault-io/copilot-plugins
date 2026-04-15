## huttle-ai

> Huttle AI development rules for React, Vite, Supabase, Tailwind CSS, and modern web development practices


You are an expert developer proficient in JavaScript, React (Vite), Supabase, Tailwind CSS, React Router DOM, React Context API, Stripe (with subscription model), and modern web development practices.

## Code Style and Structure

- Write concise, technical JavaScript/JSX code with accurate examples.
- Use functional and declarative programming patterns; avoid classes.
- Prefer iteration and modularization over code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., `isLoading`, `hasError`, `userData`).
- Structure files with exported components, subcomponents, helpers, static content, and constants.
- Favor named exports for components and functions.
- Use lowercase with dashes for directory names (e.g., `components/auth-wizard`).
- Group related imports: React imports first, then third-party, then local imports.

## JavaScript and React Patterns

- Use JavaScript for all code; prefer `const` and `let` over `var`.
- Implement functional components with clear prop destructuring.
- Use React hooks (`useState`, `useEffect`, `useContext`, `useMemo`, `useCallback`) appropriately.
- Prefer composition over inheritance.
- Use arrow functions for event handlers and callbacks.
- Use the `function` keyword for pure functions and component declarations.

## Syntax and Formatting

- Write declarative JSX with clear and readable structure.
- Use template literals for string interpolation.
- Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements.
- Use optional chaining (`?.`) and nullish coalescing (`??`) for safe property access.
- Prefer early returns to avoid deep nesting.

## UI and Styling

- Use Tailwind CSS for all styling and UI components.
- Implement responsive design with a mobile-first approach using Tailwind breakpoints.
- Use Tailwind utility classes; avoid inline styles unless necessary.
- Leverage Tailwind's custom theme colors (`huttle-primary`, `huttle-primary-dark`, `huttle-primary-light`).
- Use Lucide React for icons consistently throughout the app.
- Implement consistent spacing, typography, and color schemes using Tailwind utilities.
- Use Tailwind animations (`fadeIn`, `slideUp`) for smooth transitions.

## State Management

- Use React Context API for global state management (AuthContext, BrandContext, SubscriptionContext, etc.).
- Minimize the use of `useEffect` and `setState`; favor derived state and memoization when possible.
- Use `useMemo` and `useCallback` to optimize expensive computations and prevent unnecessary re-renders.
- Keep component state local when possible; lift state only when needed.
- Use custom hooks to encapsulate reusable stateful logic.

## Routing and Navigation

- Use React Router DOM for client-side routing.
- Implement protected routes using `ProtectedRoute` component pattern.
- Use `Navigate` component for programmatic navigation.
- Handle route parameters and query strings appropriately.
- Implement lazy loading for route components using `React.lazy()` and `Suspense`.

## Error Handling and Validation

- Prioritize error handling and edge cases.
- Handle errors and edge cases at the beginning of functions.
- Use early returns for error conditions to avoid deep nesting.
- Implement proper error logging with `console.error` for debugging.
- Display user-friendly error messages using toast notifications.
- Always check for `error` before accessing `data` in Supabase queries.
- Handle Supabase error codes gracefully (e.g., `PGRST116` for "no rows found" is not always an error).

## Performance Optimization

- Use dynamic imports (`React.lazy()`) for code splitting and route-based code splitting.
- Implement lazy loading for non-critical components.
- Use `useMemo` for expensive computations.
- Use `useCallback` for event handlers passed to child components.
- Optimize images: use appropriate formats, include size data, and implement lazy loading.
- Minimize bundle size by importing only what's needed from libraries.

## Backend and Database (Supabase)

- Use Supabase for backend services, including authentication, database interactions, and file storage.
- Follow Supabase best practices for security and performance:
  - Always wrap `auth.uid()` and `auth.jwt()` calls in `(select ...)` for RLS performance: `(select auth.uid()) = user_id`
  - Use `SET search_path = ''` for trigger functions to prevent search_path injection
  - Use `SET search_path = public` for SECURITY DEFINER functions
  - Handle `PGRST116` (no rows found) gracefully - it's not always an error
  - Use `.maybeSingle()` instead of `.single()` when a row might not exist
- Always include `.eq('user_id', userId)` in user-scoped queries for RLS.
- Use signed URLs for private bucket files; never expose private bucket URLs directly.
- Cache signed URLs client-side but respect expiration times.
- Use `.upsert()` with conflict resolution for idempotent operations.
- Default to `TIERS.FREE` when subscription lookup fails.
- Track feature usage in `user_activity` table with feature name and metadata.
- Validate storage limits before uploads using `checkStorageLimit()`.

## Stripe Integration and Subscription Model

- Implement Stripe for payment processing and subscription management.
- Use Stripe's Customer Portal for subscription management.
- Implement webhook handlers for Stripe events (e.g., subscription created, updated, or cancelled).
- Ensure proper error handling and security measures for Stripe integration.
- Sync subscription status with user data in Supabase.
- Use subscription tiers (`free`, `essentials`, `pro`) consistently throughout the app.
- Check feature access using `canAccessFeature()` helper function.

## Project Structure

- Follow the established project structure:
  - `src/components/` - Reusable UI components
  - `src/pages/` - Page components (routes)
  - `src/context/` - React Context providers
  - `src/config/` - Configuration files (Supabase, n8n)
  - `src/services/` - API service functions
  - `src/utils/` - Utility functions and helpers
  - `src/hooks/` - Custom React hooks
  - `src/layouts/` - Layout components
  - `api/` - Vercel serverless functions
- Use environment variables with `import.meta.env.VITE_*` prefix for Vite.
- Follow patterns for environment-specific configurations.

## Key Conventions

- Use descriptive and meaningful commit messages.
- Ensure code is clean, well-documented with JSDoc comments for complex functions.
- Implement error handling and logging consistently across the application.
- Use consistent naming: camelCase for variables/functions, PascalCase for components.
- Group related functionality together (e.g., all Supabase functions in `config/supabase.js`).

## Follow Official Documentation

- Adhere to the official documentation for each technology used.
- For React, focus on hooks, context API, and modern patterns.
- For Supabase, follow RLS best practices and performance guidelines.
- For Tailwind CSS, use utility-first approach and leverage custom theme configuration.
- Stay updated with the latest best practices and updates, especially for React 19, Supabase, and Vite.

## Output Expectations

- **Code Examples**: Provide code snippets that align with the guidelines above.
- **Explanations**: Include brief explanations to clarify complex implementations when necessary.
- **Clarity and Correctness**: Ensure all code is clear, correct, and ready for use in a production environment.
- **Best Practices**: Demonstrate adherence to best practices in performance, security, and maintainability.
- **Supabase Patterns**: Always follow the Supabase RLS performance patterns and error handling conventions established in the codebase.---
alwaysApply: true
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/huttleai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/huttleai)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
