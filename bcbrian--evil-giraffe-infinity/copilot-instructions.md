## evil-giraffe-infinity

> You are an expert in React Router v7, Supabase, TailwindCSS, and TypeScript,

You are an expert in React Router v7, Supabase, TailwindCSS, and TypeScript,
focusing on scalable web development.

**Key Principles**

- Provide clear, precise React Router v7 and TypeScript examples.
- Apply immutability and pure functions where applicable.
- Favor route modules and nested layouts for composition and modularity.
- Use meaningful variable names (e.g., `isAuthenticated`, `userRole`).
- Always use kebab-case for file names (e.g., `user-profile.tsx`).
- Prefer named exports for loaders, actions, and components.

**TypeScript & React Router v7**

- Define data structures with interfaces for type safety.
- Avoid the `any` type, fully utilize TypeScript's type system.
- Organize files: imports, loaders/actions, component logic.
- Use template strings for multi-line literals.
- Utilize optional chaining and nullish coalescing.
- Use nested layouts and dynamic routes where applicable.
- Leverage loaders for efficient server-side rendering and data fetching.
- Use `useFetcher` and `useLoaderData` for seamless data management between
  client and server.

**File Naming Conventions**

- `*.tsx` for React components
- `*.ts` for utilities, types, and configurations
- `root.tsx` for the root layout
- All files use kebab-case.

**Code Style**

- Use single quotes for string literals.
- Indent with 2 spaces.
- Ensure clean code with no trailing whitespace.
- Use `const` for immutable variables.
- Use template strings for string interpolation.

**React Router v7-Specific Guidelines**

- Use `<Link>` for navigation, avoiding full page reloads.
- Implement loaders and actions for server-side data loading and mutations.
- Ensure accessibility with semantic HTML and ARIA labels.
- Leverage route-based loading, error boundaries, and catch boundaries.
- Use the `useFetcher` hook for non-blocking data updates.
- Cache and optimize resource loading where applicable to improve performance.

**Import Order**

1. React Router v7 core modules
2. React and other core libraries
3. Third-party packages
4. Application-specific imports
5. Environment-specific imports
6. Relative path imports

**Error Handling and Validation**

- Implement error boundaries for catching unexpected errors.
- Use custom error handling within loaders and actions.
- Validate user input on both client and server using formData or JSON.

**Testing**

- Use `@testing-library/react` for component testing.
- Write tests for loaders and actions ensuring data correctness.
- Mock fetch requests and responses where applicable.

**Performance Optimization**

- Prefetch routes using `<Link prefetch="intent">` for faster navigation.
- Defer non-essential JavaScript using `<Scripts defer />`.
- Optimize nested layouts to minimize re-rendering.
- Use React Router v7's built-in caching and data revalidation to optimize
  performance.

**Security**

- Prevent XSS by sanitizing user-generated content.
- Use React Router v7's CSRF protection for form submissions.
- Handle sensitive data on the server, never expose in client code.

**Key Conventions**

- Use React Router v7's loaders and actions to handle server-side logic.
- Focus on reusability and modularity across routes and components.
- Follow React Router v7’s best practices for file structure and data fetching.
- Optimize for performance and accessibility.

**Reference** Refer to React Router v7’s official documentation for best
practices in Routes, Loaders, and Actions.

React Router v7 is a successor to Remix v2. There are some similarities, but
there are also some differences. Mainly the imports are just from react-router
now. do not replace react-router, imports with remix-run, imports that is
incorrect. Do not do that.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bcbrian) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
