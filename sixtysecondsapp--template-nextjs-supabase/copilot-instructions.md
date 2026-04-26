## template-nextjs-supabase

> Server Components by default and client component boundaries

## Rendering Model

- Default to Server Components (RSC). Do not add `'use client'` unless necessary.
- Client Components are only for:
  - Interactivity and event handlers
  - React hooks such as `useState`, `useEffect`, `useContext`
  - Access to browser-only APIs

### Client component rules

- Use the directive exactly as `'use client'` at the top of the file.
- Keep client components as small, leaf components. Do not convert an entire page if only a button needs state.
- Avoid passing large props trees into client components. Prefer small, serializable props from server parents.

### Server data fetching

- Fetch in RSC using Supabase server client utilities (see Supabase rule).
- Avoid creating client components just to fetch data.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SixtySecondsApp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
