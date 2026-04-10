## powder-meter

> - Next.js App Router lives in `src/app`; `layout.tsx` wires global styles, `page.tsx` renders the snow forecast shell, and `globals.css` holds global styles.

# Repository Guidelines

## Project Structure & Module Organization

- Next.js App Router lives in `src/app`; `layout.tsx` wires global styles, `page.tsx` renders the snow forecast shell, and `globals.css` holds global styles.
- Reusable UI sits in `src/components`; mark interactive pieces with `'use client'` (e.g., `CustomChart.tsx` uses MUI X Charts).
- Data fetching and server actions belong in `src/data`; `getWeatherData.tsx` pulls NOAA snowfall data with a 10s revalidate window and the required user agent header.
- Keep feature-specific assets near their feature; prefer co-locating helpers beside the component that consumes them.

## Build, Test, and Development Commands

- `npm run dev` — start the local dev server at `http://localhost:3000`.
- `npm run build` — production build for CI or preview.
- `npm run start` — serve the built app.
- `npm run lint` — Next/ESLint checks; run before pushing to catch style and safety issues.

## Coding Style & Naming Conventions

- TypeScript-first; default to server components, add `'use client'` only when browser APIs or client-only libraries (MUI X Charts) are needed.
- Components and React files use PascalCase filenames (`CustomChart.tsx`); helpers and server utilities use camelCase (`getWeatherData`).
- Never use `useMemo`, `useCallback`, or `React.memo()`. React Compiler automatically applies the optimal memoization, ensuring the app only re-renders when necessary.
- Favor concise functions, async/await for data access, and keep fetch headers/caching explicit.
- Use MUI components + `sx` for layout/styling; keep global styles minimal and scoped through `globals.css` when necessary.
- Follow ESLint defaults (run `npm run lint`); keep imports ordered by origin (React/Next → third-party → local).

## Testing Guidelines

- No tests exist
- For now, verify changes by running eslint, use MCP if available
- When testing data code, capture a fixture of the NOAA response to avoid network flakiness.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/barrar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/barrar)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
