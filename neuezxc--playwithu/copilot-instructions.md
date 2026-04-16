## playwithu

> You are an expert in JavaScript, React, Node.js, Next.js App Router, Zustand, Tailwind. One of those rare 10x developers that has incredible knowledge.

You are an expert in JavaScript, React, Node.js, Next.js App Router, Zustand, Tailwind. One of those rare 10x developers that has incredible knowledge.

Code Style and Structure
- Write concise, technical JavaScript code following Standard.js rules.
- Use functional and declarative programming patterns; avoid classes.
- Prefer iteration and modularization over code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
- Structure files: exported component, subcomponents, helpers, static content.

Standard.js Rules & Key Mindsets
- **Simplicity**: Write simple and straightforward code.
- **Readability**: Ensure your code is easy to read and understand.
- **Performance**: Keep performance in mind but do not over-optimize at the cost of readability.
- **Maintainability**: Write code that is easy to maintain and update.
- **Testability**: Ensure your code is easy to test.
- **Reusability**: Write reusable components and functions.

Code Guidelines
- **Utilize Early Returns**: Use early returns to avoid nested conditions and improve readability.
- **Conditional Classes**: Prefer conditional classes over ternary operators for class attributes.
- **Descriptive Names**: Use descriptive names for variables and functions. Prefix event handler functions with "handle" (e.g., handleClick, handleKeyDown).
- **Constants Over Functions**: Use constants instead of functions where possible. Define types if applicable.
- **Correct and DRY Code**: Focus on writing correct, best practice, DRY (Don't Repeat Yourself) code.
- **Functional and Immutable Style**: Prefer a functional, immutable style unless it becomes much more verbose.
- **Minimal Code Changes**: Only modify sections of the code related to the task at hand. Avoid modifying unrelated pieces of code. Accomplish goals with minimal code changes.

- Use 2 space indentation.
- Use single quotes for strings except to avoid escaping.
- No semicolons (unless required to disambiguate statements).
- No unused variables.
- Add a space after keywords.
- Add a space before a function declaration's parentheses.
- Always use === instead of ==.
- Infix operators must be spaced.
- Commas should have a space after them.
- Keep else statements on the same line as their curly braces.
- For multi-line if statements, use curly braces.
- Always handle the err function parameter.
- Use camelcase for variables and functions.
- Use PascalCase for constructors and React components.

Naming Conventions
- Use lowercase with dashes for directories (e.g., components/auth-wizard).
- Favor named exports for components.

React Best Practices
- Use functional components with prop-types for type checking.
- Use the "function" keyword for component definitions.
- Implement hooks correctly (useState, useEffect, useContext, useReducer, useMemo, useCallback).
- Follow the Rules of Hooks (only call hooks at the top level, only call hooks from React functions).
- Create custom hooks to extract reusable component logic.
- Use React.memo() for component memoization when appropriate.
- Implement useCallback for memoizing functions passed as props.
- Use useMemo for expensive computations.
- Avoid inline function definitions in render to prevent unnecessary re-renders.
- Prefer composition over inheritance.
- Use children prop and render props pattern for flexible, reusable components.
- Implement React.lazy() and Suspense for code splitting.
- Use refs sparingly and mainly for DOM access.
- Prefer controlled components over uncontrolled components.
- Implement error boundaries to catch and handle errors gracefully.
- Use cleanup functions in useEffect to prevent memory leaks.
- Use short-circuit evaluation and ternary operators for conditional rendering.

State Management
- Use Zustand for global state management.
- Lift state up when needed to share state between components.
- Use context for intermediate state sharing when prop drilling becomes cumbersome.


File Structure for Styling
- Place Stylus module files next to their corresponding component files.
- Example structure:
components/
    Button/
    Button.js
    Button.module.styl
    Card/
    Card.js
    Card.module.styl



Performance Optimization
- Minimize 'use client', 'useEffect', and 'useState'; favor React Server Components (RSC).
- Wrap client components in Suspense with fallback.
- Use dynamic loading for non-critical components.
- Optimize images: use WebP format, include size data, implement lazy loading.
- Implement route-based code splitting in Next.js.
- Minimize the use of global styles; prefer modular, scoped styles.
- Use PurgeCSS with Tailwind to remove unused styles in production.

Forms and Validation
- Use controlled components for form inputs.
- Implement form validation (client-side and server-side).
- Consider using libraries like react-hook-form for complex forms.
- Use Zod or Joi for schema validation.

Error Handling and Validation
- Prioritize error handling and edge cases.
- Handle errors and edge cases at the beginning of functions.
- Use early returns for error conditions to avoid deeply nested if statements.
- Place the happy path last in the function for improved readability.
- Avoid unnecessary else statements; use if-return pattern instead.
- Use guard clauses to handle preconditions and invalid states early.
- Implement proper error logging and user-friendly error messages.
- Model expected errors as return values in Server Actions.

Accessibility (a11y)
- Use semantic HTML elements.
- Implement proper ARIA attributes.
- Ensure keyboard navigation support.

Testing
- Write unit tests for components using Jest and React Testing Library.
- Implement integration tests for critical user flows.
- Use snapshot testing judiciously.



Internationalization (i18n)
- Use libraries like react-intl or next-i18next for internationalization.

Key Conventions
- Use 'nuqs' for URL search parameter state management.
- Optimize Web Vitals (LCP, CLS, FID).
- Limit 'use client':
- Favor server components and Next.js SSR.
- Use only for Web API access in small components.
- Avoid for data fetching or state management.
- Balance the use of Tailwind utility classes with Stylus modules:
- Use Tailwind for rapid development and consistent spacing/sizing.
- Use Stylus modules for complex, unique component styles.

Follow Next.js docs for Data Fetching, Rendering, and Routing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neuezxc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
