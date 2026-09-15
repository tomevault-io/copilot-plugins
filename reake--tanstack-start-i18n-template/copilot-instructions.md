## tanstack-start-i18n-template

> - `src/` holds application code.

# Repository Guidelines

## Project Structure & Module Organization

- `src/` holds application code.
  - `src/routes/` — TanStack Start file-based routes. Route groups live in parentheses (`(pages)`), the optional locale segment is `{-$locale}`, and the root document is `__root.tsx`.
  - `src/components/` — `layout/` (header, footer), `landing/` (landing page sections), `blog/`, `legal/` (long-form page layout), `ui/` (shadcn-style primitives).
  - `src/lib/` — `i18n.ts` (locales, message loading, path helpers), `seo.ts` (meta tags, JSON-LD), `blog.ts` (content queries), `crawl-cleanup.ts`, `utils.ts`.
  - `src/messages/<locale>/*.json` — message catalogs. `en` is the default and the fallback.
  - `src/generated/` — build output for blog modules. Never edit by hand; it is gitignored.
- `content/blogs/` holds MDX posts named `slug.<locale>.mdx`.
- `public/` holds static assets plus the generated `sitemap.xml` and `robots.txt`.
- `scripts/` contains `blog-data-generator.mjs` and `generate-sitemap.mjs`.
- `dist/` is the Cloudflare Pages build output.
- `.output/` is the Cloudflare Workers build output when `CF_DEPLOY_TARGET=workers`.

## Build, Test, and Development Commands

- `pnpm dev`: Vite dev server on `http://localhost:3000`.
- `pnpm build`: production build and sitemap generation for Cloudflare Pages.
- `pnpm exec wrangler --cwd dist pages dev`: preview the Cloudflare Pages build.
- `pnpm run deploy` / `pnpm run deploy:pages`: build and deploy Pages.
- `pnpm run deploy:workers`: build and deploy Workers.
- `pnpm sitemap:generate`: regenerate `public/sitemap.xml` and `public/robots.txt`.
- `pnpm lint`, `pnpm check-types`, `pnpm check`: ESLint, TypeScript, and read-only format+lint+types checks.
- `pnpm test` / `pnpm test:watch` / `pnpm test:coverage`: Vitest.

## Coding Style & Naming Conventions

- TypeScript + React with ESM modules.
- Prettier enforces 2-space indentation, semicolons, double quotes, 90-char print width, trailing commas, LF endings, and organized imports (see `.prettierrc`).
- ESLint uses TypeScript, React Hooks, and TanStack Query/Router rules; keep lint error-free (warnings are acceptable).
- Prefer kebab-case file names (e.g. `site-header.tsx`, `blog-post-page.tsx`).
- Site identity belongs in `src/site.config.ts`; do not hardcode names or URLs.

## Internationalization Rules

- The default locale (`en`) is never prefixed. Use `withLocalePath` / `stripLocaleFromPath` from `src/lib/i18n.ts` instead of building locale URLs by hand.
- Every user-visible string belongs in `src/messages`. Add a namespace to `MESSAGE_NAMESPACES` before adding a JSON file.
- Locales are discovered from `src/messages/*/common.json`. Adding a folder plus a `NATIVE_LANGUAGE_LABELS` entry is enough for routing, hreflang, and the sitemap.

## SEO Rules

- Every route's `head()` must emit `generateSeoMeta` plus `generateCanonicalLink` for its localized path.
- The root document emits `hreflang` alternates and `x-default`; blog posts only advertise locales that exist.
- Structured data helpers live in `src/lib/seo.ts`. Prefer adding a helper there over inline JSON-LD.
- `sitemap.xml` and `robots.txt` are generated — never edit them manually.

## Testing Guidelines

- Tests are co-located with code and named `*.test.ts` or `*.test.tsx`.
- Use Testing Library + Vitest. Focus on utilities and routing logic; add a test when you change `src/lib/i18n.ts`, `src/lib/seo.ts`, or a shared component.

## Commit & Pull Request Guidelines

- Commits use short, single-line summaries, often in Chinese. Keep them concise and focused.
- PRs should describe the change, link related issues, and include screenshots for UI changes. Note the commands you ran (e.g. `pnpm test`, `pnpm check`).

## Configuration & Secrets

- No environment variables are required. If you add one, document it in the README and add it to `.env.example`.
- Never commit `.env`, credentials, or generated build output.

---
> Source: [reake/tanstack-start-i18n-template](https://github.com/reake/tanstack-start-i18n-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
