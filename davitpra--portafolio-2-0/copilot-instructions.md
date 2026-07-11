## portafolio-2-0

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start development server (http://localhost:3000)
npm run build    # Production build
npm run start    # Start production server
npm run lint     # Run ESLint (next/core-web-vitals)
```

No test framework is configured.

## Environment

Requires a `.env.local` file:
```
NEXT_PUBLIC_SITE_URL=https://example.com
```

## Architecture

Next.js 14 App Router portfolio site. All source code lives in `src/`.

**Routing (`src/app/`):** Standard App Router conventions. Pages: `/`, `/about`, `/articles`, `/articles/[slug]`, `/projects`, `/experience`, `/studies`, `/thank-you`, `/feed.xml` (RSS route handler).

**Blog articles (`src/app/articles/[slug]/page.mdx`):** Each article is a directory containing a `page.mdx` file. Articles export a named `article` object and a `metadata` object as frontmatter, then default-export a wrapper using `ArticleLayout`. The `src/lib/articles.js` module uses `fast-glob` to load and sort all articles at build time — this is used by the homepage (top 3), the articles listing page, and the RSS feed.

MDX article structure:
```js
export const article = { author, date, title, description }
export const metadata = { title, description }
export default (props) => <ArticleLayout article={article} {...props} />
```

**Styling:** Tailwind CSS v4 via `@tailwindcss/postcss`. Custom typography config in `src/styles/typography.js`. Prism syntax highlighting via `rehype-prism-plus` with styles in `src/styles/prism.css`. Dark mode is managed by `next-themes` with system preference detection.

**Path alias:** `@/*` maps to `./src/*`.

**Key components (`src/components/`):**
- `Layout.jsx` — root wrapper (Header + Footer + content)
- `Container.jsx` — responsive max-width wrapper with `ContainerOuter`/`ContainerInner` subcomponents
- `ArticleLayout.jsx` — article page template with back-navigation using `AppContext`
- `Card.jsx` — flexible card with `.Link`, `.Title`, `.Description`, `.Cta`, `.Eyebrow` subcomponents
- `Prose.jsx` — typography container for MDX content (dark mode aware)
- `SimpleLayout.jsx` — standard page layout (title + intro + children)

**Providers (`src/app/providers.jsx`):** Client-side — `ThemeProvider` (next-themes), `AppContext` (tracks previous pathname for article back-button), `ThemeWatcher` (syncs system theme changes).

**MDX config (`next.config.mjs`):** `@next/mdx` with `remark-gfm` and `rehype-prism-plus`. MDX files are treated as pages.

**Prettier:** single quotes, no semicolons, Tailwind class sorting (`prettier-plugin-tailwindcss`).

---
> Source: [davitpra/Portafolio-2.0](https://github.com/davitpra/Portafolio-2.0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
