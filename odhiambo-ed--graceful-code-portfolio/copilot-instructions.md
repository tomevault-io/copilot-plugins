## graceful-code-portfolio

> when you are considering Core Project Standards


# Core Project Standards

- Use Next.js 15 with App Router for all routing and rendering.
- Write all code in TypeScript with strict mode enabled (`tsconfig.json` settings: `"strict": true`).
- Follow Airbnb TypeScript style guide for code formatting (e.g., 2-space indentation, semicolons).
- Use Tailwind CSS for styling, avoiding inline styles unless necessary.
- Organize files in a modular structure:
  - Pages: `app/[route]/page.tsx`
  - Components: `src/components/[ComponentName]/[ComponentName].tsx`
  - Server actions: `src/actions/[action].ts`
  - Utilities: `src/lib/utils.ts`
  - Supabase queries: `src/lib/supabase/queries.ts`
- Prefer functional components and React hooks over class components.
- Use server components by default; use "use client" only when client-side interactivity is required.
- Export components and functions with named exports (e.g., `export function ComponentName`).
- Include JSDoc comments for all public functions and components.
- Ensure error handling with try-catch for async operations and proper logging.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/odhiambo-ed) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
