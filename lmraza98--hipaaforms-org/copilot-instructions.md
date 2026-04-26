## hipaaforms-org

> description: "File and folder conventions for Next.js 15 App Router"

---
description: "File and folder conventions for Next.js 15 App Router"
globs: src/app/**
---
# Next.js App Router Structure
- Organize code under the `src/app/` directory. Each folder under `app` creates a route segment:contentReference[oaicite:9]{index=9}.
- Every route folder needs a `page.tsx` (or `page.jsx/ts`) file to be publicly accessible:contentReference[oaicite:10]{index=10}. Example: `src/app/dashboard/page.tsx` serves `/dashboard`.
- Use `layout.tsx` files in route folders to wrap child pages (e.g. common headers/footers):contentReference[oaicite:11]{index=11}. Each layout defines UI that persists across its sub-routes.
- Place API endpoints as `route.ts` (or `route.js/ts`) in the App Router (e.g. `src/app/api/submit/route.ts` for a POST handler).
- Use Next.js dynamic route syntax: `[param]` for parameters and `[...slug]` for catch-all segments:contentReference[oaicite:12]{index=12}. For example, `src/app/forms/[id]/page.tsx` matches `/forms/{id}`.
- Keep static assets in `public/`. Use `next/image` or `<img>` for images. Shared UI components can live in `src/components/` or under `app/components/`, imported into pages/layouts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lmraza98) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
