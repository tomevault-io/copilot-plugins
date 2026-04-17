## cm-nextjs-boilerplate

> - Prefer functional components.

# React / Next.js (Web)

## Components
- Prefer functional components.
- Keep components small and focused; extract reusable UI into `src/common/components/`.

## Routing (Pages Router)
- Use the Pages Router under `src/pages/`.
- Use `next/router` APIs (`useRouter`) for navigation in Pages Router.
- Avoid `next/navigation` in Pages Router code.
- Only use `next/navigation` in App Router code (e.g. `src/app-disabled/`).
- Do not add `'use client'` directives unless working in the App Router.

## Hooks
- Follow Rules of Hooks; keep hook calls at the top level.

## Conditional rendering
- Prefer early returns for loading/error states.

## Context
- Use a separate hook to create context value.
- Use return type of this hook to define context value type.

## Project structure
- Keep pages and API routes in `src/pages/` (API routes in `src/pages/api/`).
- Put cross-cutting concerns in `src/common/`:
  - `src/common/components/` for shared components
  - `src/common/layouts/` for shared page layouts
  - `src/common/context/` for context providers + hooks
  - `src/common/constants/` for constants/enums
  - `src/common/styles/` for global/theme styles
- Put feature modules in `src/modules/`.

## Apollo
- Use loading state returned by the Apollo hook instead of defining separate loading state.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/commutatus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
