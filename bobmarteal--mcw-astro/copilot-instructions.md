## mcw-astro

> Marteal Cycle Works marketing site. Custom bicycle paint/build portfolio with a bikes content collection.

# CLAUDE.md — MCW-Astro

## Project Overview
Marteal Cycle Works marketing site. Custom bicycle paint/build portfolio with a bikes content collection.

## Tech Stack
- **Framework**: Astro 5.x (currently 5.15.9), static output (no adapter)
- **Deployment**: CloudCannon (migrated from Netlify) — build output is `dist/`, config in `cloudcannon.config.yml`
- **Styling**: SASS/SCSS — global partials in `src/styles/sass/`, component-scoped styles in each `.astro` file
- **Type checking**: TypeScript (strict mode, `astro/tsconfigs/strict`)
- **Icons**: `astro-icon` 1.x
- **Fonts**: Google Fonts (Open Sans, Shantell Sans) loaded in `src/layouts/Head.astro`
- **PostCSS**: autoprefixer via `postcss.config.cjs`
- No React, Vue, or other UI framework — Astro-only

## Folder Conventions
```
src/
  assets/            # Source images processed by astro:assets. Organized by bike slug (e.g., assets/fuji/)
  components/        # Reusable .astro UI components — PascalCase filenames
  content/           # Content collection entries (markdown)
  content.config.ts  # Zod schemas for collections (Astro 5 location — NOT src/content/config.ts)
  layouts/           # Page wrapper components: Head, MainPageWrapper, SiteHeader, SiteFooter
  pages/             # File-based routes
  styles/            # Global SASS — style.scss imports base + layout partials only
  env.d.ts           # Astro type references
public/              # Unprocessed static assets (SVGs, favicon only — no images)
```

## Content Collections
- **bikes** — defined in `src/content.config.ts` using Astro 5 `glob()` loader
  - Required: `name` (string), `type` (array of "Build" | "Paint"), `cover` (image — build-time optimized)
  - Optional: `description` (string, max 5000), `lede` (string, max 200), `gallery` (array of filename strings)
  - Gallery images are filename strings resolved at runtime via `import.meta.glob` in `[slug].astro`, not build-time validated

## Path Aliases (tsconfig.json)
- `@/*` → `src/*`
- `@components/*` → `src/components/*`
- `@layouts/*` → `src/layouts/*`
- `@ui/*` → `src/components/ui/*`

Aliases are picked up automatically by Vite — no need to duplicate in `astro.config.mjs`.

## Integrations
- `astro-icon` — SVG icon system
- `@astrojs/check` — TypeScript checking runs as part of build (`astro check && astro build`)

## Observed Preferences
- Component styles live in the `.astro` file's `<style lang="scss">` block, not in global sass partials
- Global SASS partials are for resets, vars, utilities, and layout systems only
- Use Astro's `<Image>` component for all processed images (never raw `<img>` for content images)
- No client-side JS frameworks — vanilla JS only for interactivity
- CSS custom properties used for theming: `--yellow`, `--teal`, `--blue`, `--red`
- Slots used for component content flexibility (see `Lede.astro` for default slot fallback pattern)

## Coding Conventions
<!-- TODO: Fill in — e.g., naming patterns, TypeScript interface conventions in frontmatter, preferred slot patterns -->

## Deployment Target
CloudCannon, static output. No adapter in `astro.config.mjs` — Astro's default `output: 'static'` mode. CloudCannon builds via `npm run build` and serves `dist/`.

## Performance Goals
<!-- TODO: clarify — Core Web Vitals targets, image format preferences (avif/webp), lazy loading rules -->

## Known Issues / Tech Debt
- jQuery loaded in `Head.astro` but not used by any component — candidate for removal
- `getCollection("bikes")` is called independently in both `index.astro` and `Skewgallery.astro`
- `src/components/ImageGallery.astro` exists but is not imported anywhere
- `src/styles/sass/nav/` and `src/styles/sass/components/` partials are not imported in `style.scss`
- No Open Graph or social meta tags in `Head.astro`
- `description` prop is not passed from pages to `MainPageWrapper`, leaving the meta description empty

---
> Source: [bobmarteal/MCW-Astro](https://github.com/bobmarteal/MCW-Astro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
