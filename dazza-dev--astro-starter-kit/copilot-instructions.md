## astro-starter-kit

> Starter kit for building websites with **Astro 6** and **WordPress** as a headless CMS using **GraphQL**. Supports **i18n** with Spanish (default) and English via Polylang.

# CLAUDE.md - Project Guidelines

## Project Overview

Starter kit for building websites with **Astro 6** and **WordPress** as a headless CMS using **GraphQL**. Supports **i18n** with Spanish (default) and English via Polylang.

## Tech Stack

- **Framework**: Astro 6 (SSR mode, `output: "server"`)
- **Styling**: Tailwind CSS 4 (via `@tailwindcss/vite` plugin)
- **Deployment**: Vercel adapter (`@astrojs/vercel`) by default
- **CMS**: WordPress + WPGraphQL + Polylang
- **Package Manager**: pnpm (v10.33.2) - always use `pnpm`, never npm/yarn
- **Language**: TypeScript (strict mode)
- **Linting**: ESLint + Prettier

## Commands

```bash
pnpm dev          # Start dev server
pnpm build        # Production build
pnpm preview      # Preview production build (deploy runs on Vercel)
pnpm lint         # Run ESLint
pnpm lint:fix     # Fix ESLint issues
pnpm format       # Format with Prettier
pnpm format:check # Check formatting
pnpm type-check   # Run astro check (TypeScript)
```

## Project Structure

```
src/
  components/       # Reusable UI components (.astro)
    BlogCard.astro      # Blog post card for listings
    Breadcrumb.astro    # Breadcrumb navigation
    ImagePlaceholder.astro  # Fallback SVG when no image available
    PageHeader.astro    # Dark band: eyebrow + title + subtitle
    PageBody.astro      # WordPress page body (featured image + sanitized content)
    Lightbox.astro      # Image/YouTube modal (astro-icon); one per page, data-* triggers
    HeadlessForm.astro  # Renders a WPForms form headlessly (needs SW - WPForms GraphQL)
  i18n/             # i18n utilities (I18nUtils.ts)
  lib/              # Data layer & utilities
    graphql.ts      # GraphQL client (timeout + retry; optional {lang} for Polylang)
    blog.ts         # Blog post queries (WordPress native posts + Polylang)
    pages.ts        # WordPress Pages by slug (extend with your plugin's fields)
    menu.ts         # WordPress menus by location, Polylang-aware, normalized URLs
    siteSettings.ts # Site title/logo + ACF Options (contact, socials); cached per lang
    forms.ts        # WPForms schema via GraphQL (SW - WPForms GraphQL plugin)
    sitemap.ts      # Dynamic sitemap generation (queries WP for all slugs)
    url.ts          # URL builder helpers (buildUrl, isCurrentPage)
    utils.ts        # HTML/SVG sanitization (sanitize-html)
  locales/          # Translation strings, locale config, pageSlugs (Locales.ts)
  pages/            # Astro file-based routing
    index.astro     # Root redirect (302 + Vary: Accept-Language)
    404.astro       # Self-contained error page (no Layout dependency)
    500.astro       # Self-contained error page (no Layout dependency)
    sitemap-index.xml.ts  # Sitemap index endpoint
    sitemap-0.xml.ts      # Main sitemap with all URLs + hreflang
    [lang]/               # Locale-prefixed dynamic routes
      index.astro              # Home page
      [page].astro             # Generic WordPress page (resolved via pageSlugs)
      blog/index.astro         # Blog listing
      blog/[slug].astro        # Blog detail
      blog/feed.ts             # RSS feed (per language)
  theme/
    layouts/Layout.astro  # Main HTML layout (SEO, hreflang, OG/Twitter, JSON-LD, RSS)
    views/                # Page-level view components (HomeView, PageView, Blog*)
    styles/
      global.css          # Tailwind import, theme styles
      wordpress-content.css  # Styles for WordPress HTML content (.wp-content)
  types/
    blog.ts         # Post types and GraphQL responses
```

## Architecture Patterns

### Data Flow

Pages (`src/pages/`) fetch data and pass it to Views (`src/theme/views/`). Views compose Layout + Components. Components are pure presentational.

```
Page (data fetching) -> View (composition) -> Layout + Components
```

### GraphQL Client (`src/lib/graphql.ts`)

- Central `graphqlQuery<T>()` function with 10s timeout and AbortController
- All data modules (blog, sitemap) use this client
- `normalizeLanguageFilter()` converts locale to uppercase for WPGraphQL Polylang enum
- Error handling: each module catches errors, logs with `console.error`, returns empty/null fallback
- Uses `astro:env/server` for type-safe env var access

### i18n System

- **Locales.ts**: Single source of truth for all translation strings, locale config
- Both compile-time (`satisfies`) and runtime validation ensure ES/EN keys stay in sync
- **I18nUtils.ts**: `getLangFromUrl()`, `useTranslations()`, `getOtherLang()`, re-exports from `url.ts`
- All routes are prefixed with locale: `/{lang}/`, `/{lang}/blog/`, etc.
- `trailingSlash: "always"` - all URLs end with `/`

### URL Building

- Always use `buildUrl(lang, section?, slug?)` from `src/lib/url.ts`
- Sections type: `"blog"` (extend as needed)
- Slugs are URI-encoded via `encodeURIComponent`

### Security

- All WordPress HTML content is sanitized via `sanitize-html` before rendering
- Three sanitization functions: `stripHtml()`, `sanitizeHtml()`, `sanitizeSvg()`
- SVG sanitization preserves case sensitivity (`lowerCaseTags: false`)
- Environment variables use Astro's `envField` with `context: "server"` and `access: "secret"`

### Dynamic Sitemap (`src/lib/sitemap.ts`)

- Custom implementation (replaces `@astrojs/sitemap` which doesn't work with SSR dynamic routes)
- Queries WordPress for all post slugs per language
- Generates XML with `xhtml:link` hreflang alternates
- Endpoints: `/sitemap-index.xml` -> `/sitemap-0.xml`

### RSS Feed

- Per-language RSS 2.0 feed with `atom:link` self-reference
- URLs: `/es/blog/feed/`, `/en/blog/feed/`
- Autodiscovery via `<link rel="alternate" type="application/rss+xml">` in Layout

### Error Pages

- 404 and 500 are self-contained (no Layout imports) to avoid cascading failures
- Use dynamic imports with try/catch fallback for i18n
- Inline styles guarantee display even if CSS fails

### Root Redirect

- 302 redirect based on `Accept-Language` header
- Includes `Vary: Accept-Language` and `Cache-Control: no-store` to prevent proxy caching

## TypeScript Path Aliases

```
@components/* -> src/components/*
@lib/*        -> src/lib/*
@layouts/*    -> src/theme/layouts/*
@views/*      -> src/theme/views/*
@i18n/*       -> src/i18n/*
@locales/*    -> src/locales/*
```

## Code Conventions

### General

- TypeScript strict mode with `noUnusedLocals`, `noUnusedParameters`, `noFallthroughCasesInSwitch`
- No `any` allowed (`@typescript-eslint/no-explicit-any: "error"`)
- Unused vars must be prefixed with `_`
- `no-console` is warn-level (only `console.warn` and `console.error` allowed)
- Use double quotes, semicolons, trailing commas, 2-space indent, 120 char print width

### Astro Components

- All components define `interface Props` for type safety
- Use `getLangFromUrl(Astro.url)` + `useTranslations(lang)` for i18n in components
- Error pages (404, 500) are self-contained (no Layout imports) to avoid cascading failures

### Styling

- Tailwind CSS 4 with custom theme tokens via CSS variables
- WordPress content uses `.wp-content` class with dedicated stylesheet

### Data Fetching in Pages

- Validate `lang` param with `isSupportedLocale()` before any data fetching
- Rewrite to `/404` if locale is invalid
- Use `Promise.all()` for parallel data fetching when multiple queries are needed
- Wrap in try/catch and rewrite to `/500` on error
- Check for null results and rewrite to `/404`

```astro
const { lang } = Astro.params;
if (!lang || !isSupportedLocale(lang)) return Astro.rewrite("/404");

try {
  data = await fetchData(lang);
} catch {
  return Astro.rewrite("/500");
}
```

## Environment Variables

- `WORDPRESS_GRAPHQL_URL` (required, secret): WordPress GraphQL endpoint
- `WORDPRESS_IMAGE_DOMAINS` (build-time, via `process.env`): Comma-separated image domains for `astro:assets`

## WordPress Integration Modules

These are wired but expect specific WordPress plugins / configuration. Adjust field names to each project.

- **Pages** (`lib/pages.ts` + `[lang]/[page].astro` + `PageView`/`PageHeader`/`PageBody`): renders native WP pages **fully dynamically** — any published page resolves by its URL slug, so editors add pages in WordPress with zero code changes. Polylang `translations` are queried to build correct hreflang/language-switch URLs when a page's slug differs per language. Only standard Page fields are queried — extend the interface + query when your site plugin (e.g. **SW - Site Skeleton**) adds custom fields: `swEyebrow`, `swSubtitle`, `swSection`, `swGalleries`, `swCardSections`, `swFormId`. `pageSlugs` in `Locales.ts` is optional (only for linking from code to a specific page).
- **Menus** (`lib/menu.ts`): needs **WPGraphQL for Polylang**. Menu location per language is `PRIMARY` (default lang) / `PRIMARY___EN` (others) — do NOT also pass a `language` where-arg. Set `OWN_HOSTS` so absolute links to your domain resolve internally.
- **Site settings** (`lib/siteSettings.ts`): `generalSettings.title` + a `siteLogoUrl` root field (from your site plugin) + an **ACF Options Page** exposed as `siteSettings.siteSettingsFields`. ACF names fields from labels — rename to match. Cached per language.
- **Forms** (`lib/forms.ts` + `HeadlessForm.astro`): needs the **SW - WPForms GraphQL** plugin (schema via `wpForm(id)`, submit via `POST /wp-json/sw/v1/forms/{id}/submit`). Handles compound name/address, native date pickers, and a drawn-signature field (WPForms field with CSS class `signature` → canvas → saved as a media image). Give a select/radio "Show Values" or the label is used as the value.
- **Icons** (`astro-icon` + `@iconify-json/lucide`): `import { Icon } from "astro-icon/components"` then `<Icon name="lucide:x" />`. Inline SVG, no external requests. Add more sets with `pnpm add @iconify-json/<set>`.

## Gotchas & Lessons Learned

Hard-won constraints — check these before debugging:

- **Polylang + ACF Options don't switch language via GraphQL.** The built-in `language` where-arg has no effect on an Options Page. Pass `graphqlQuery(query, vars, { lang })`; it appends `?lang=xx` so a site plugin can set Polylang's current language (`graphql_before_execute` hook).
- **The HTML sanitizer strips inline styles**, keeping only `class`. Any design driven from Gutenberg must use **CSS classes** (Advanced → Additional CSS class), never the block toolbar's inline styles. Add the matching rule to `wordpress-content.css`.
- **`<script>` in `.astro` is bundled; `<script define:vars>` is NOT** (can't use `import`). Pass data via `data-*` attributes and read them in a bundled script.
- **`astro check` does not catch missing image imports.** A page can type-check and still 500 at runtime with `ImageNotFound`. Boot the dev server and curl the affected routes after touching image imports.
- **Avoid regex literals in `.astro` frontmatter** — `/^\/[^/]+/` and friends can break the frontmatter parser. Use string methods (`split`/`join`) instead.
- **Taxonomy queries need `hideEmpty: false`** or terms with no posts disappear.
- **WPGraphQL requires "Post name" permalinks** or `/graphql` 404s.
- **WPForms select/radio choices** store an empty string value when "Show Values" is off — the plugin falls back to the label, but be aware the submitted value is then the label text.
- **Comments explain the non-obvious "why", not the obvious "what".** No iteration/changelog noise left in code.

## Adding New Languages

1. Add locale to `astro.config.mjs` `i18n.locales` array
2. Update `SupportedLocale` type, `supportedLocales` array, and `ui` object in `src/locales/Locales.ts` (all keys must match)
3. Add `localeHreflang` mapping in `src/locales/Locales.ts`
4. Add `localeConfig` entry (flag + label) in `src/i18n/I18nUtils.ts`
5. Add `channelMeta` entry in `src/pages/[lang]/blog/feed.ts`
6. Add error page text in `src/pages/404.astro` and `src/pages/500.astro`
7. The `[lang]` dynamic route handles all locales automatically - no new pages needed
8. Configure the language in Polylang within WordPress

## Adding New Content Types

1. Create a data module in `src/lib/` following the pattern of `blog.ts`
2. Define TypeScript interfaces for the GraphQL response in `src/types/`
3. Use `graphqlQuery<T>()` with proper error handling
4. Add page routes in `src/pages/[lang]/`
5. Create view component in `src/theme/views/`
6. Add translation keys to both `es` and `en` in `src/locales/Locales.ts` (keys must match exactly)
7. Add the new section to `Section` type in `src/lib/url.ts`
8. Add the new slugs to `src/lib/sitemap.ts` for sitemap inclusion

## Adding Translation Keys

1. Add the key to **both** `es` and `en` objects in `src/locales/Locales.ts`
2. The compile-time `satisfies` check and runtime validation will catch mismatches
3. Use the key via `t("your.key")` after calling `useTranslations(lang)`

---
> Source: [dazza-dev/astro-starter-kit](https://github.com/dazza-dev/astro-starter-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
