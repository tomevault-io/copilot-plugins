## personal-website

> TanStack Start powers routing in `src/routes`, where each file exports `createFileRoute` with co-located loaders via `createServerFn`. Shared UI primitives live in `src/components/ui`, while higher-level widgets sit beside feature code under `src/components`. Markdown-driven content (blog posts, metadata) resides in `src/content`, parsed through helpers in `src/lib`. Global styles, Tailwind setup, and CSS variables are defined in `src/styles/app.css`. Generated assets such as `src/routeTree.gen.ts` should be treated as read-only.

# Repository Guidelines

## Project Architecture & Directories

TanStack Start powers routing in `src/routes`, where each file exports `createFileRoute` with co-located loaders via `createServerFn`. Shared UI primitives live in `src/components/ui`, while higher-level widgets sit beside feature code under `src/components`. Markdown-driven content (blog posts, metadata) resides in `src/content`, parsed through helpers in `src/lib`. Global styles, Tailwind setup, and CSS variables are defined in `src/styles/app.css`. Generated assets such as `src/routeTree.gen.ts` should be treated as read-only.

## Rendering & Data Flow Patterns

Routes hydrate on the server first, then seamlessly hand off to the client. Fetch external data inside `createServerFn` handlers to keep secrets server-side and return serializable payloads. For client-side revalidation, call `router.invalidate()` or leverage loader dependencies. Keep long-running effects out of render and favor derived data selectors.

## UI System & Styling Conventions

Use Shadcn/ui components as the base layer, extending them with Tailwind utility classes. For conditional styling, compose class strings with the `cn` helper from `@/lib/utils`. Typography pairings are baked into components; prefer prop-driven variants over ad-hoc classes. Respect the design tokens exposed as OKLCH CSS variables (`bg-background`, `text-muted-foreground`, etc.) to maintain light/dark parity.

## Tooling & Development Commands

Run `npm run dev` for the Vite dev server with SSR and hot reloading at http://localhost:3000. `npm run build` generates the production bundle, while `npm run start` serves that build for verification. The repo currently has no automated tests—additions should introduce Vitest + React Testing Library and update the `npm test` script before integrating with CI.

## Content & Asset Workflow

Blog entries use Markdown with frontmatter in `src/content`; keep slugs in sync with filenames (`blog.$slug.tsx`). Optimized images ship from `public/` or are imported directly for bundling; run additions through image compression to protect Core Web Vitals. Environment variables such as `GITHUB_TOKEN` or `VITE_GITHUB_TOKEN` enable GitHub API features—define them locally without committing secrets.

---
> Source: [urdadx/personal-website](https://github.com/urdadx/personal-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
