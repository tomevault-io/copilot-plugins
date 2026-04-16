## frisbee-festival

> This is a static SvelteKit 5 website for the Frisbee Festival München using Tailwind CSS 4. The site is multilingual (German as source locale, English translation) and uses the Wuchale i18n framework for translations.

## Project Overview

This is a static SvelteKit 5 website for the Frisbee Festival München using Tailwind CSS 4. The site is multilingual (German as source locale, English translation) and uses the Wuchale i18n framework for translations.

## Architecture

### Internationalization (i18n)

The site uses **Wuchale** for translations:

- Source locale: `de` (German)
- Available locales: `de`, `en` (defined in `src/locales/data.js`)
- Translation files: `src/locales/de.po` and `src/locales/en.po` (PO/gettext format)
- Compiled translations are generated in `src/locales/.wuchale/`

**Key i18n files:**

- `src/hooks.server.ts`: Server-side locale context setup using `runWithLocale()`, pre-loads all catalogs at startup
- `src/routes/[[locale=lang]]/+layout.ts`: Client-side locale loading with `loadLocale()`
- `src/params/lang.ts`: Route matcher for locale parameter validation
- `src/lib/locale.ts`: Utility to extract locale from URL pathname

### Routing Structure

Routes use SvelteKit's optional locale parameter pattern: `[[locale=lang]]`

- Default locale (German) is served at root: `/`
- English pages are prefixed: `/en/`
- All pages under `src/routes/[[locale=lang]]/` support both locales
- Special routes without locale: `/health`, `/sitemap.xml`

### Static Site Generation

- Adapter: `@sveltejs/adapter-static`
- All routes are prerendered (`prerender = true` in layout)
- Build output: `build/` directory
- Configured for static hosting (no server-side rendering at runtime)

### UI Components

- Component library: **shadcn-svelte** see [shadcn-svelte](https://www.shadcn-svelte.com/llms.txt)
- Custom components in `src/lib/components/`
- Always use shadcn-svelte primitives for consistency and accessibility, ask to install new components if needed
- Styling: Tailwind CSS 4 with custom utilities (`tw-animate-css`, `tailwind-variants`, `tailwind-merge`)

### Build Configuration

- **Vite plugins**: Tailwind CSS, SvelteKit, Wuchale
- **Svelte 5**: Uses runes and modern syntax
- **Node version**: >=22.12.0

## Adding Translations

1. Add translation keys to `src/locales/de.po` (source locale)
2. Run dev server or build - Wuchale will auto-compile translations
3. Translated content appears in `.wuchale/` directory (do not edit manually)
4. Use translation functions in components (refer to Wuchale docs)

## Important Notes

- The dev server is already running (per user config)
- German (`de`) is the default/source locale and should have empty URL prefix
- All translation compilation is handled automatically by the Wuchale Vite plugin
- UI components follow the shadcn-svelte pattern with bits-ui primitives

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fluid-movement) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
