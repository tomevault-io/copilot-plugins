## senflix-marketing

> This repository contains the marketing site for the African Video Platform.

# General context for GitHub Copilot in this repository

This repository contains the marketing site for the African Video Platform.

## Primary stack

- Next.js 16 App Router
- React 19
- TypeScript
- Tailwind CSS v4
- shadcn-compatible component structure
- Vercel deployment

## Architecture expectations

1. Keep App Router files focused on routing, layout, metadata, and page composition.
2. Reuse the existing structure under `src/app`, `src/components`, `src/hooks`, and `src/lib`.
3. Preserve the current bilingual content model and theme behavior.
4. Keep the visual language intentional and consistent with the existing landing page direction.
5. Prefer small, composable sections and shared UI primitives over page-local duplication.
6. Treat the installed Next.js 16 behavior as authoritative over older defaults.

## Boundaries

Respect feature boundaries strictly.

1. Do not bury page content logic inside low-level UI components.
2. Do not duplicate content, theme, or layout helpers if a shared component or utility already owns them.
3. Keep route-level metadata and SEO-sensitive concerns in App Router files or shared utilities that clearly own them.
4. Keep shared presentation logic in `src/components` or `src/lib` instead of scattering one-off helpers across pages.
5. Preserve existing deployment assumptions for Vercel unless the task explicitly changes deployment behavior.

## High-risk areas

Be extra careful in:

- `src/app`
- localized landing page copy
- theme switching behavior
- SEO metadata and social preview content
- external links and conversion-focused call-to-action sections

For these areas, preserve content intent, routing stability, hydration safety, and production build behavior.

## Validation

After marketing changes, prefer targeted validation first, then broader validation when needed:

- `npm run lint`
- `npm run type-check`
- `npm run build`

---
> Source: [Nebur242/senflix-marketing](https://github.com/Nebur242/senflix-marketing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
