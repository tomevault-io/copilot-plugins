## decoder22-github-io

> You are an expert developer in TypeScript, React, Shadcn UI, Radix UI, and styled-components.

You are an expert developer in TypeScript, React, Shadcn UI, Radix UI, and styled-components.

---

Key Principles
- Write concise, technical responses with accurate TypeScript examples.
- Use functional, declarative programming. Avoid classes.
- Prefer iteration and modularization over duplication.
- Use descriptive variable names with auxiliary verbs (e.g., `isLoading`, `hasError`).
- Use lowercase with dashes for directories (e.g., `components/auth-wizard`).
- Favor named exports for components.
- Use the Receive an Object, Return an Object (RORO) pattern.

---

JavaScript/TypeScript
- Use `function` keyword for pure functions. Omit semicolons.
- Use TypeScript for all code. Prefer interfaces over types.
- File structure:
  - Exported component
  - Subcomponents
  - Helpers
  - Static content
  - Types
- Always include curly braces in conditional statements.
- For single-line statements in conditionals, include curly braces.
- For simple conditional statements (e.g., `if (condition) doSomething()`) make sure to include curly braces. Do not leave the logic on one line.

---

Error Handling and Validation
- Prioritize error handling and edge cases:
  - Handle errors and edge cases at the beginning of functions.
  - Use early returns for error conditions to avoid deeply nested `if` statements.
  - Place the happy path last in the function for improved readability.
  - Avoid unnecessary `else` statements; use `if-return` pattern instead.
  - Use guard clauses to handle preconditions and invalid states early.
  - Implement proper error logging and user-friendly error messages.
  - Consider using custom error types or error factories for consistent error handling.
  - In the error factory, have error messages print in red to the console and warning messages print in yellow to the console.

---

React
- Use functional components and TypeScript interfaces.
- Use declarative JSX.
- Use `function` (not `const`) for components.
- Avoid Next.js-specific concepts (like App Router or `use client`) since you're not using Next.js.
- Use **styled-components** for styling instead of Tailwind CSS.
- Implement responsive design using **styled-components** media queries.
- Place static content and interfaces at the file end.
- Use content variables for static content outside render functions.
- Minimize `useEffect` and `setState`. Favor state hooks directly in functional components.
- Use Zod for form validation.
- Wrap client components in Suspense with a fallback.
- Use dynamic loading for non-critical components.
- Optimize images: WebP format, size data, lazy loading.
- Model expected errors as return values: Avoid using try/catch for expected errors.
- Implement error boundaries for unexpected errors: Implement error boundaries using `error.tsx` and `global-error.tsx` files.

---

Naming Conventions
- **Booleans**: Use auxiliary verbs such as 'does', 'has', 'is', and 'should' (e.g., `isDisabled`, `hasError`).
- **Filenames**: Use lowercase with dash separators (e.g., `auth-wizard.tsx`).
- **File extensions**: Use `.config.ts`, `.test.ts`, `.context.tsx`, `.type.ts`, `.hook.ts` as appropriate.

---

Component Structure
- Break down components into smaller parts with minimal props.
- Suggest micro folder structure for components.
- Use composition to build complex components.
- Follow the order:
  - Component declaration
  - Styled components (if any)
  - TypeScript types

---

Styling
- **Do not use Tailwind CSS**: Use **styled-components** for styling instead.
- Use **styled-components** for theme-based styles and utility classes.
- For **component variants**, you can use a utility function or an object-based approach to handle different styles in `styled-components`.
- Implement responsive design using **styled-components** media queries.

---

Testing
- Implement unit tests for utility functions and hooks.
- Use integration tests for complex components and pages.
- Implement end-to-end tests for critical user flows.

---

Documentation
- Provide clear and concise comments for complex logic.
- Use **JSDoc comments** for functions and components to improve IDE intellisense.
- Keep the **README** files up-to-date with setup instructions and project overview.

---

Accessibility
- Ensure interfaces are keyboard navigable.
- Implement proper ARIA labels and roles for components.
- Ensure color contrast ratios meet WCAG standards for readability.

---

Additional Notes
- Since you're not working with a backend or database, focus solely on the front-end. 
- Avoid external system integrations such as **Supabase** or **GraphQL**.
- Focus on creating a smooth experience using **React** and **styled-components**.
- Ensure all components are modular, easily testable, and maintainable.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Decoder22) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
