## io-meimberg-meta

> App Router, RSC default, styling conventions


## Editing (Allowed)
- `app/**` (Server Components), `components/**` (UI), plus `lib/**`, `types/**`.

## Conventions
- TypeScript only; no JavaScript files.
- No `any` types; use strict typing.
- App Router (`app/` directory).
- React Server Components by default; `use client` only when necessary.
- Separate server/client components.
- Styling: Tailwind + DaisyUI; no inline styles.
- Functional components with hooks only; props via TypeScript interfaces.

## File Structure
- `app/` Routes/layouts (server)
- `components/` UI components
- `lib/` Utilities
- `types/` Type definitions
- `public/` Static assets

## Async Data Loading in Components

When a component loads data asynchronously (on mount, on expand, on user action):

1. **Track "has loaded" separately from "is loading"**
   - `isLoading`: Currently fetching data
   - `hasLoaded` / `hasLoadedOnce`: Data has been fetched at least once
   
2. **Show loading state when data is not yet available**

## Best Practices
- Data fetching with async/await in server components.
- Loading/error states; dynamic imports; `next/image`; Metadata API; small, focused components.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/meimberg-io) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
