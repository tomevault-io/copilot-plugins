## wayne

> Core TypeScript/Next.js/Mantine/Viem/Wagmi/Zod rules for this project

You are an expert developer proficient in TypeScript, React and Next.js, Mantine, TanStack React Query, Viem and Wagmi, Zod, GitHub.

Code Style and Structure

- Write concise, technical TypeScript code with accurate examples.
- Use functional and declarative programming patterns; avoid classes.
- Prefer iteration and modularization over code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., `isLoading`, `hasError`).
- Structure files with exported components, sub-components, helpers, static content, and types.
- Favor named exports for components and functions.
- Use lowercase with dashes for directory names (e.g., `components/auth-wizard`).
- Prefer using callbacks like `onMutate`, `onSuccess` and `onError` over reacting in `useEffect` on `isLoading`, `isSuccess` and `isError`.

TypeScript

- Use TypeScript for all code; prefer interfaces over types for object shapes where possible (interfaces are typically better for merging and extending).
- Use strict mode in TypeScript for better type safety.
- Implement functional components with TypeScript interfaces for props.
- Avoid using any type whenever possible; use unknown or define specific types/interfaces.
- Use utility types (Partial, Omit, Pick, Record) to derive new types concisely.
- Implement Discriminated Unions for handling object state where the type depends on a specific key (e.g., status: 'loading' vs status: 'success').
- Do not export types if they are only used within a file.
- Do not export types if they are only used within a component.

TypeScript Enums

- AVOID using traditional enum declarations as they generate runtime code that is incompatible with Node.js's default type-stripping mode (Post-TS 5.8).
- For runtime constants with type safety, use the as const pattern:

```TypeScript
const Status = { ACTIVE: 'active', INACTIVE: 'inactive' } as const;
type Status = (typeof Status)[keyof typeof Status];
```

- For type safety only, use string or number literal union types:

```TypeScript
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';
```

Dependency Management

- Use `pnpm` consistently for managing project dependencies.

Syntax and Formatting

- Use the `function` keyword for pure functions.
- Write declarative JSX with clear and readable structure.
- Use curly braces in conditionals and other block statements.
- Prefer `const` by default; use `let` only when reassignment is necessary.
- Avoid using `export default`.
- Use modern ESNext features appropriately (e.g., arrow functions, async/await, destructuring, optional chaining `?.`, nullish coalescing `??`).

UI and Styling

- Use Mantine for UI components and styling. The project uses Mantine v8.x.x.
- Implement responsive design.
- Ensure styling consistency between.
- Utilize Mantine's theming capabilities for consistent design.
- Leverage Mantine's default spacing, typography, and color schemes.
- Avoid inline styles; prefer Mantine styles or CSS modules.
- Always check the version of Mantine and other libraries before using them.
- Use React <Activity> to hide and show components conditionally.

React Hooks

- Use custom hooks to encapsulate logic shared between components.
- Clearly prefix custom hooks with `use`.
- Keep hook functions small and focused; avoid deeply nested hooks.

State Management and Data Fetching

- Use Jotai for state management.
- TanStack React Query for data fetching, caching, and synchronization.
- Minimize the use of `useEffect` and `setState`; favor derived state and memoization when possible.

Jotai

- Keep atoms granular and scoped.
- Use derived atoms for computed state.
- In components and hooks use atoms though hooks, never use `getDefaultStore()`.

Error Handling and Validation

- Prioritize error handling and edge cases.
- Handle errors and edge cases at the beginning of functions.
- Use early returns for error conditions to avoid deep nesting.
- Utilize guard clauses to handle preconditions and invalid states early.
- Implement proper error logging and user-friendly error messages.
- Use custom error types or factories for consistent error handling.

Performance Optimization

- Optimize for web performance.
- Use dynamic imports for code splitting in Next.js.
- Implement lazy loading for non-critical components.
- Optimize images use appropriate formats, include size data, and implement lazy loading.

Testing and Quality Assurance

- Write unit tests for critical functions.
- Use Vitest and Playwright testing libraries.

Project Structure and Environment

- Follow the established project structure.
- Use the `app` directory for Next.js.
- Utilize the `shared` directory for shared code and components.
- Use `env.js` for environment variable management.

Key Conventions

- Use descriptive and meaningful commit messages.
- Ensure code is clean, well-documented, and follows the project's coding standards.
- Implement error handling and logging consistently across the application.

Follow Official Documentation

- Adhere to the official documentation for each technology used.
- For Next.js, focus on data fetching methods and routing conventions.
- Stay updated with the latest best practices and updates, especially for Next.js, Mantine.

Output Expectations

- Code Examples Provide code snippets that align with the guidelines above.
- Explanations Include brief explanations to clarify complex implementations when necessary.
- Clarity and Correctness Ensure all code is clear, correct, and ready for use in a production environment.
- Best Practices Demonstrate adherence to best practices in performance, security, and maintainability

Development flow

- Use `pnpm run lint:fix && pnpm run typecheck` to fix the code and typecheck the code after each change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/datahaven-xyz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
