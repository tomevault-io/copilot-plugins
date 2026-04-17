## pausalniprihodi

> - This repository powers the public marketing site for `pausalni-prihodi.com` and the embedded Payload CMS admin/API.

## Project Purpose
- This repository powers the public marketing site for `pausalni-prihodi.com` and the embedded Payload CMS admin/API.
- The business app itself lives on `app.pausalni-prihodi.com`; landing CTAs should link there.

## Core Stack
- Next.js App Router (v15), TypeScript, Tailwind CSS v4, shadcn/ui primitives.
- Payload CMS v3 with PostgreSQL via `@payloadcms/db-postgres`.
- Rich text must use Payload Lexical fields and frontend rendering helpers.

## Architecture Rules
- Public pages live under `src/app/(frontend)/`.
- Payload admin and API routes live under `src/app/(payload)/`.
- Payload schema and runtime config live in `src/payload.config.ts`, `src/collections/`, `src/globals/`.
- Prefer server components by default; add `"use client"` only for interactive UI.
- Keep side effects in route handlers, server actions, hooks, or dedicated server utilities.

## Content Modeling Rules
- Landing sections should be modeled as Payload Globals (Hero, Features, FAQ, etc.).
- Reusable or list-like entities should be Collections (Users, Media, ContactSubmissions, LegalPages).
- Legal pages (`privacy`, `terms`, `cookies`) are CMS-driven and rendered from Payload content.
- Use clear Serbian field labels where editor-facing text benefits from localization.

## SEO Rules
- Every public page must export metadata with canonical URL and meaningful title/description.
- Home page should include JSON-LD where applicable (Organization, WebSite, SoftwareApplication, FAQPage).
- Keep `<html lang="sr">` in root layout.
- `src/app/sitemap.ts` and `src/app/robots.ts` must reflect public routes and disallow `/admin`.

## Revalidation and Caching
- Default ISR for landing/legal routes is 12h (`revalidate = 43200`).
- Payload `afterChange` hooks must call `revalidatePath` for impacted routes.
- Site-wide setting changes should invalidate shared layout dependencies.

## UI and Styling
- Follow design tokens and Tailwind utility patterns from the landing design system.
- Avoid hardcoded magic colors; prefer semantic classes or shared token values.
- Maintain consistent heading hierarchy and spacing rhythm across sections.

## Language and Copy
- User-facing copy is Serbian by default unless a feature explicitly requires another language.
- Domain and email references should use `.com` variants unless config-driven from CMS.

## Maintenance
- Remove dead code and dependencies when replacing legacy modules.
- Keep files focused; split large components into section/component modules.
- Add small comments only when logic is non-obvious.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cabarkapa99) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
