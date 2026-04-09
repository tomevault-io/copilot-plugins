## adalert-io-app

> You are an expert in TypeScript, Node.js, React, Next.js App Router, Tailwind CSS, Shadcn UI, Radix UI, and Zustand. Follow best practices in code structure, naming, styling, and performance.

You are an expert in TypeScript, Node.js, React, Next.js App Router, Tailwind CSS, Shadcn UI, Radix UI, and Zustand. Follow best practices in code structure, naming, styling, and performance.

---

Code Style and Structure
- Use functional, declarative React components; avoid class components.
- Use descriptive variable names with auxiliary verbs (e.g., `isLoading`, `hasError`, `canSubmit`).
- Prefer early returns to reduce nesting and improve readability.
- Use the RORO (Receive an Object, Return an Object) pattern for complex function arguments.
- Minimize repetition by modularizing logic (e.g., extract helpers and utility functions).

---

Component Structure & File Organization
- Each component should live in its own folder:
  - `[Component].tsx`: main file
  - `helpers.ts`: utility functions
  - `types.ts`: local interfaces and types
  - `index.ts`: re-export for cleaner imports
- Structure top-level directories by feature or domain (e.g., `features/auth`, `features/dashboard`).
- Use PascalCase for file and component names.

---

Directory and File Naming
- Use kebab-case for directories (e.g., `components/auth-wizard`).
- Use PascalCase for component files and exported symbols.
- Favor named exports over default exports for better tooling and readability.

---

TypeScript Usage
- All code must be written in TypeScript.
- Prefer `interface` over `type` for defining object shapes.
- Avoid `enum`; use object literals or `Record<string, string>` for mappings.
- Use strict typing; never use `any`.

---

Naming Conventions
- Event handlers should be prefixed with `handle` (e.g., `handleClick`, `handleSubmit`).
- Boolean variables must start with auxiliary verbs (`is`, `has`, `should`, `can`).
- Hook names must start with `use` (e.g., `useUserStore`, `useFetchData`).

---

UI and Styling
- Use Shadcn UI for all form elements, modals, and standard components.
- Prefer Radix UI icons; avoid third-party icon packs unless justified.
- Style using Tailwind CSS. Default spacing: `p-4`, `gap-4`, `rounded-2xl`, `shadow-md`.
- Use `text-sm`, `text-base`, `text-lg` for font sizes; maintain visual consistency.
- Implement responsive design using Tailwind's mobile-first utilities (`sm:`, `md:`, `lg:`).
- Ensure accessibility: use semantic HTML, `aria-` attributes, and focus states.

---

State Management
- Use Zustand for global state management:
  - Keep stores small and domain-focused.
  - Define state shape using interfaces.
  - Persist state only when needed (e.g., auth token).
- Do not store server-fetched data in global state; use it locally or with React Query.

---

Error Handling & Guard Clauses
- Use guard clauses to exit early from functions or components.
- Define and reuse custom error types for predictable error handling.
- Always check for null, undefined, or unexpected values before proceeding.
- Wrap async functions with try/catch and log context-aware error messages.

---

Performance Optimization
- Limit `'use client'` usage. Use React Server Components (RSC) whenever possible.
- Avoid unnecessary `useEffect` or `useState` calls.
- Dynamically import non-critical components.
- Use `Suspense` with fallbacks for client-side rendering.
- Optimize images: use `next/image`, prefer WebP, and include width/height.

---

Data Fetching
- Use `getServerSideProps`, `getStaticProps`, or server components for data fetching.
- Use React Query for caching and background updates.
- Do not fetch data in client components unless interactive.
- Avoid prop drilling; co-locate data where it's used.

---

Follow official [Next.js documentation](https://nextjs.org/docs) for routing, rendering strategies, and best practices.



Get-ChildItem -Recurse -Exclude node_modules, .next, .vscode, .git | Compress-Archive -DestinationPath ../tfs-refactor-check.zip -Force

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adalert-io)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adalert-io)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
