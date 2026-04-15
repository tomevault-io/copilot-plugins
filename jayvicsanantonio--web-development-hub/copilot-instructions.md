## web-development-hub

> These rules define the standards and practices for the Windsurf AI project, ensuring consistency, maintainability, and use of the latest stable versions of all core libraries and tools.


# Code Style Guide

These rules define the standards and practices for the Windsurf AI project, ensuring consistency, maintainability, and use of the latest stable versions of all core libraries and tools.

---

## General Code Style & Formatting

- **Follow the [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)** for all JavaScript/TypeScript code, including React components and modules.
- **React component file names must use PascalCase** (e.g., `UserCard.tsx`, not `user-card.tsx`).
- **Prefer named exports** for all components and utilities.

---

## Project Structure & Architecture

- **Use Next.js (`v15.4.0` or latest stable)** for the application framework.
  - Follow Next.js conventions for file/folder structure.
  - Use the **App Router** for routing and layouts.
  - **Correctly determine server vs. client components**:
    - Use server components by default for data-fetching and logic-heavy pages.
    - Use client components only when interactivity, hooks, or browser APIs are required.

---

## Styling & UI

- **Tailwind CSS `v4.0`** is required for all styling. NEVER use v3.
  - Configure Tailwind according to the new v4 CSS-first configuration approach.
  - Use Tailwind utility classes in JSX/TSX for styling elements.
- **Shadcn UI (latest CLI and components)** for prebuilt UI components.
  - Use the new CLI (`npx shadcn init` and `npx shadcn add`) for adding and updating components.
  - Ensure Tailwind and Shadcn UI are integrated (CLI will handle config updates).
  - Prefer Shadcn UI components for common UI patterns (e.g., Drawer, Pagination, Carousel).

---

## Forms

- **React Hook Form `v7.57.0`** for form state management and validation.
  - Use the latest APIs and features for form control and error handling.
- **Zod `v3.25.67`** for schema validation.
  - Define all validation schemas with Zod.
  - Integrate Zod schemas with React Hook Form for type-safe, declarative validation.

---

## State Management & Logic

- **Use React Context** for global or shared state.
  - Avoid third-party state management libraries unless a clear need arises.
  - Keep context providers minimal and focused.

---

## Additional Requirements

- **Keep all dependencies up-to-date** with the latest stable versions, especially for critical libraries listed above.
- **Comments should be used sparingly** and only for:
  - Documenting complex logic that isn't self-explanatory
  - Explaining workarounds or non-obvious solutions
  - Documenting library version deviations or special handling
  - API documentation (JSDoc for public interfaces)
- **Write and maintain comprehensive tests** for all components, hooks, and business logic.
- **Use pnpm** as the project's package manager. **DO NOT** use npm.

---

## Summary Table of Core Libraries/Tools

| Purpose          | Library/Tool    | Latest Stable Version (as of June 24, 2025) |
| ---------------- | --------------- | ------------------------------------------- |
| Framework        | Next.js         | 15.4.0                                      |
| Styling          | Tailwind CSS    | 4.0                                         |
| UI Components    | Shadcn UI       | Latest CLI (Aug 2024+)                      |
| Forms            | React Hook Form | 7.57.0                                      |
| Validation       | Zod             | 3.25.67                                     |
| State Management | React Context   | (built-in)                                  |

---

## References

- [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
- [Next.js v15.4.0](https://nextjs.org/)
- [Tailwind CSS v4.0](https://tailwindcss.com/)
- [Shadcn UI](https://ui.shadcn.com/)
- [React Hook Form v7.57.0](https://react-hook-form.com/)
- [Zod v3.25.67](https://zod.dev/)

---

Adhering to these rules will ensure your project is modern, maintainable, and leverages the best practices and latest stable technologies available as of mid-2025.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jayvicsanantonio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
