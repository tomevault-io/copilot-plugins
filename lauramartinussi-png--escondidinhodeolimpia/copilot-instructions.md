## escondidinhodeolimpia

> This is a Netlify-hosted TanStack Start marketing site scaffolded for deployment.

# Agent Notes

This is a Netlify-hosted TanStack Start marketing site scaffolded for deployment.

## Architecture

- `src/routes/__root.tsx` defines the root HTML shell, document metadata, and global CSS import.
- `src/routes/index.tsx` renders the home page and links into product detail routes.
- `src/routes/products/$productId.tsx` renders each product detail page from the shared product catalog.
- `src/data/products.ts` stores the current static product data used by both list and detail pages.
- `src/router.tsx` creates the TanStack Router instance.
- `src/styles.css` contains global styles and imports Tailwind CSS.
- `public/` contains static assets served directly by Netlify.
- `netlify.toml` defines the Netlify build command and publish directory.

## Technology

- TanStack Start and TanStack Router for the React application and file-based routing.
- React 19 for UI rendering.
- Vite for local development and production bundling.
- Tailwind CSS 4 for styling utilities.
- Netlify Vite plugin for deployment compatibility with TanStack Start.
- TypeScript with strict settings.

## Conventions

- Keep route files under `src/routes/` and use TanStack Router file-route helpers.
- Keep shared content data in `src/data/` unless it needs persistence.
- Use Tailwind utilities for component-level styling and `src/styles.css` for global rules.
- Prefer the existing `@/` path alias for imports from `src/`.
- Do not commit generated build output such as `dist/`, `.netlify/`, or route generation artifacts.

## Netlify Notes

The project is configured to build with `vite build` and publish `dist/client`. The platform validation pipeline is responsible for installing dependencies and running the production build.

---
> Source: [lauramartinussi-png/escondidinhodeolimpia](https://github.com/lauramartinussi-png/escondidinhodeolimpia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
