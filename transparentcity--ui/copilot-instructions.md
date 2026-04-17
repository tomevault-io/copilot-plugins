## ui

> Before building new templates, pages, or components **always** check:

# transparentcity-ui - Cursor Rules
#
# Frontend for TransparentCity / TransparentSF v2.0+
# Tech stack: Next.js App Router, React, TypeScript
#
# Goals:
# - Keep UI thin and focused on presentation + UX
# - Treat FastAPI backends as the single source of truth
# - Share mental model with TransparentSF backend rules (modular, typed, well-documented)

## Branding & Design References

Before building new templates, pages, or components **always** check:

1. `docs/BRAND_KIT.md` in this repo – frontend brand kit (favicon, logo usage,
   bracket loader patterns, quick tokens).
2. The shared Visual Design System style guide in the TransparentCity backend
   repo (`ai/static/data/Style Guide`) – canonical colors, typography, spacing,
   charts, and maps.

Match new UI to those references unless there is an explicit product reason
to diverge.

## Architecture

1. **App Router only**
   - Use `src/app` with the App Router (no `pages/` directory).
   - Co-locate route-specific components under their route where it keeps things readable.
   - Extract shared layout and UI primitives into `src/components` and `src/ui`.

2. **API Boundary**
   - All HTTP calls to backends go through small helpers in `src/lib/` (e.g. `apiClient.ts`, future `transparentSfClient.ts`).
   - Do not call `fetch` directly from random components; use a wrapper so we can change base URLs / auth in one place.
   - Prefer **server components** for data fetching where possible; use client components only for interactive pieces.

3. **Backends**
   - Default base URL is `NEXT_PUBLIC_API_BASE_URL` (e.g. TransparentCity platform).
   - If/when we talk to TransparentSF directly, add a **separate, clearly named client** (e.g. `transparentSfApiClient.ts`) rather than reusing the same base.

## Code Style

1. **TypeScript**
   - All new files should be `.ts` or `.tsx` with explicit types for props and exported helpers.
   - Avoid `any` except as a last resort; prefer proper interfaces for API responses.

2. **React / Next**
   - Prefer **server components** by default; mark client components explicitly with `"use client"`.
   - Keep components small and focused; avoid giant catch-all containers.
   - Favor composition over deep prop drilling (use small context providers where necessary).

3. **Styling**
   - Use CSS modules for route-specific styles or a simple utility-first approach (e.g. Tailwind-style utility classes in `globals.css`) – but keep a consistent pattern per file.
   - Keep color and typography tokens centralized where possible.

4. **Performance**
   - Design components to be **React Compiler friendly** (pure render, no side effects in render), but **do not rely on React Compiler yet**.
   - Avoid unnecessary client-side global state; prefer server data + local component state.

## API & Data Fetching

1. **Health & Diagnostics**
   - The `/debug/health` route should always call backend `/health` via `src/lib/apiClient.ts`.
   - If backend URL changes, update `NEXT_PUBLIC_API_BASE_URL` and `apiClient.ts` only.

2. **Error Handling**
   - All API helpers should throw rich `Error` objects; UI components are responsible for rendering a friendly message.
   - For debug/admin views, it is OK to show raw error text; for public views, prefer user-friendly copy.

3. **Auth (future)**
   - When auth is introduced, centralize token / cookie handling in `src/lib/apiClient.ts` and/or a dedicated auth client.

## Project Hygiene

1. **Folders**
   - `src/app`       – Next.js routes, layouts, and route-level components.
   - `src/components` – Shared presentational + small container components.
   - `src/lib`       – API clients, utilities, config helpers.
   - `src/types`     – Shared TypeScript types and interfaces.

2. **Tests (future)**
   - Co-locate component tests next to components when we add a test runner.

3. **Do Not**
   - Do not duplicate backend business logic in the frontend.
   - Do not build new server-rendered templates in the backends for UI; this repo owns the user and admin UX.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Transparentcity) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
