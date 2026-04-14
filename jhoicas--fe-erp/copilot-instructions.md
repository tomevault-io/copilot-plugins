## fe-erp

> - **Stack:** Next.js (App Router), TypeScript (Strict), Tailwind CSS, Radix UI/Shadcn UI.

# ERP Architecture & Development Rules

## 1. Stack & Core Principles
- **Stack:** Next.js (App Router), TypeScript (Strict), Tailwind CSS, Radix UI/Shadcn UI.
- **Architecture:** Atomic Design + Modular Features. Divide the app into `features/` (e.g., `features/inventory`, `features/billing`).
- **Performance:** Mandatory use of React Server Components (RSC) by default. Client components only for leaf nodes requiring interactivity.
- **Micro-frontends:** Treat internal modules as isolated packages. Use Shared Components for UI consistency.

## 2. Rendering & Performance (Anti-Bloat)
- **Granular Rendering:** Every complex view must be broken into small, memoized components to ensure individual re-rendering.
- **Optimized DOM:** Minimize div nesting. Use CSS Grid/Flexbox efficiently. Avoid "div-soup".
- **Loading States:** Every module must implement `loading.tsx` and Skeleton patterns for perceived speed.
- **Data Fetching:** Use TanStack Query (React Query) for client-side state and Server Actions for mutations.

## 3. Security & "Blindado" (Security First)
- **Input Sanitization:** All text inputs must be validated with **Zod** schemas. Sanitize all HTML/String inputs to prevent XSS.
- **URL Protection:** Use middleware-level RBAC (Role-Based Access Control). Never expose sensitive IDs in URLs; use UUIDs or encrypted slugs.
- **Console/Network Defense:** Disable sensitive logging in production. Implement "Server-Only" patterns for API keys and logic.
- **Injection Prevention:** Use parameterized queries (via Prisma/Drizzle) and never use `dangerouslySetInnerHTML`.
- **Form Safety:** All forms must use `react-hook-form` + `zod`.

## 4. Coding Standards (Cursor Logic)
- **TypeScript:** No `any`. Use discriminated unions for complex states.
- **Error Handling:** Use Error Boundaries per module. No silent failures.
- **Modular Exports:** Use index files (barrels) only at the feature level to prevent circular dependencies.
- **Detail:** When generating UI, ensure accessibility (ARIA labels) and high contrast.

## 5. Prevention of Infinite Loops
- Before suggesting a code change, verify that it doesn't trigger a recursive re-render in `useEffect` or circular imports between `features/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jhoicas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
