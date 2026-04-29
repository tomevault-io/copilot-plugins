## ink-ui

> This is the documentation site for **Ink UI**, built with **Astro 5**, **React 19**, **Tailwind CSS 4**.

# AGENTS.md

## Project overview

This is the documentation site for **Ink UI**, built with **Astro 5**, **React 19**, **Tailwind CSS 4**.

## Common commands

All commands use `pnpm`:

- `pnpm dev` — Start the Astro dev server.
- `pnpm build` — Run `astro build`. Required before running tests.
- `pnpm preview` — Preview the production build locally.
- `pnpm typecheck` — Run `astro check` for TypeScript validation.
- `pnpm test` — Run Vitest tests.
- `pnpm lint` — Run oxlint on `src/`.
- `pnpm fmt` — Format code with oxfmt.
- `pnpm fmt:check` — Check formatting without writing.

Linting and formatting are handled by **oxlint** and **oxfmt** (not ESLint/Prettier). Configs live in `.oxlintrc.json` and `.oxfmtrc.json`.

## High-level architecture

### Local component registry (`registry/`)

The `registry/` directory contains local UI wrappers, examples, and styles that are specific to the docs site:

- `registry/components/ui/` — Wrappers around Base UI primitives (e.g., `button.tsx`) with local styling.
- `registry/example/` — Demo functions used by `<ComponentExample demo="...">`.
- `registry/lib/utils.ts` — Local `cn()` utility.
- `registry/styles/ink.css` — Additional Tailwind theme variables.

Path aliases:

- `~/*` → `./src/*`
- `@registry/*` → `./registry/*`

### Demo and props system

In MDX docs, use these Astro components:

- `<ComponentExample demo="ButtonExample">` — Renders the demo and auto-extracts the function source code from `registry/example/**/*.tsx` using AST parsing (`src/lib/extract-demo-snippet.ts`).
- `<PropsTable component="Button" />` — Currently stubbed out (not available in generic mode).

### Markdown page generation

The custom Astro integration `markdownPages` (`src/lib/astro-markdown-pages.ts`) generates a `.md` file for every doc page at build time. These are used by the "Copy page" feature and for LLM integrations.

### Build-time defines

`astro.config.mjs` injects git and package metadata as global compile-time constants:

- `__DOCS_VERSION__`, `__BUILD_COMMIT__`, `__BUILD_COMMIT_DATE__`, `__BUILD_BRANCH__`, `__BUILD_DATE__`

These are declared in `src/build-info.d.ts`.

### Styling

- Tailwind CSS v4 is configured via the Vite plugin (`@tailwindcss/vite`).
- Global styles are in `src/styles/global.css`, which imports `registry/styles/ink.css`.
- The prose wrapper class `.ink-prose` applies Tailwind Typography overrides.
- Dark mode is toggled via the `data-mode="dark"` attribute on `<html>`.

### Testing

Tests use Vitest. Run `pnpm build` before `pnpm test`.

### Deployment

`wrangler.jsonc` configures deployment. The site URL is `https://ink-ui.vercel.app/`.

---
> Source: [anxndsgn/ink-ui](https://github.com/anxndsgn/ink-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
