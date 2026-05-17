## peppy-mandazi-b65ae9

> This project is a TanStack Start landing page for AtlasOps, a fictional launch operations platform. It is intended to deploy on Netlify and uses Netlify Forms for newsletter signups.

# AGENTS.md

This project is a TanStack Start landing page for AtlasOps, a fictional launch operations platform. It is intended to deploy on Netlify and uses Netlify Forms for newsletter signups.

## Architecture

- `src/routes/__root.tsx` defines the root HTML shell, global metadata, and imports global styles.
- `src/routes/index.tsx` contains the full landing page experience, including the hero, feature highlights, testimonials, call-to-action, footer, and newsletter form.
- `src/styles.css` defines global browser defaults, typography, selection color, and Tailwind CSS loading.
- `public/__forms.html` is a static Netlify Forms skeleton. It must stay in sync with the React newsletter form field names so Netlify can detect the form at build time.
- `public/` contains static assets served from the site root.

## Key Technologies

- TanStack Start and TanStack Router for routing and rendering.
- React 19 with TypeScript for UI logic.
- Tailwind CSS 4 for utility-first styling.
- Lucide React for interface icons.
- Netlify Forms for serverless newsletter submission handling.

## Coding Conventions

- Use TypeScript and keep strict-mode friendly types.
- Prefer file-based routes in `src/routes/`.
- Keep route components focused and colocate small page-only components inside the route file when they are not reused elsewhere.
- Use `@/` imports for source paths when importing across directories.
- Use Tailwind utilities for component styling and reserve `src/styles.css` for global defaults.
- Keep form submissions URL-encoded when posting to Netlify Forms.

## Non-Obvious Decisions

- Newsletter submissions post to `/__forms.html` rather than `/`. TanStack Start uses SSR routing, so posting to the static skeleton path ensures Netlify Forms receives the submission.
- The visible React newsletter form and the hidden static form in `public/__forms.html` both use the `newsletter` form name. Changing one requires changing the other.
- The landing page uses a product-dashboard composition instead of a stock image so the first viewport communicates the product category immediately.
- The project intentionally avoids adding a separate component library layer until there are multiple pages sharing the same UI primitives.

## Development Notes

- Do not run production build commands unless explicitly requested; automated validation handles builds.
- Use `npm run dev` for Vite development or `netlify dev` when Netlify feature emulation is needed.
- Do not commit build output or generated deployment artifacts.

---
> Source: [aaronhtg-lang/peppy-mandazi-b65ae9](https://github.com/aaronhtg-lang/peppy-mandazi-b65ae9) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
