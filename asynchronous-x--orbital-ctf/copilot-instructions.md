## nextjs

> description: This rule explains Next.js conventions and best practices for fullstack development.

---
description: This rule explains Next.js conventions and best practices for fullstack development.
globs: **/*.js,**/*.jsx,**/*.ts,**/*.tsx
alwaysApply: false
---

# Next.js rules

- Use the App Router structure with `page.tsx` files in route directories.
- Client components must be explicitly marked with `'use client'` at the top of the file.
- Use kebab-case for directory names (e.g., `components/auth-form`) and PascalCase for component files.
- Prefer named exports over default exports, i.e. `export function Button() { /* ... */ }` instead of `export default function Button() { /* ... */ }`.
- Minimize `'use client'` directives:
  - Keep most components as React Server Components (RSC)
  - Only use client components when you need interactivity and wrap in `Suspense` with fallback UI
  - Create small client component wrappers around interactive elements
- Avoid unnecessary `useState` and `useEffect` when possible:
  - Use server components for data fetching
  - Use React Server Actions for form handling

  - Use URL search params for shareable state

---
> Source: [asynchronous-x/orbital-ctf](https://github.com/asynchronous-x/orbital-ctf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
