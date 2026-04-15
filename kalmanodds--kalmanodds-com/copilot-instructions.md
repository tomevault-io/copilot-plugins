## kalmanodds-com

> Astro blog/portfolio site. Uses content collections for blog posts and projects, Tailwind for styling, React for interactive components (search).

# kalmanodds.com

Astro blog/portfolio site. Uses content collections for blog posts and projects, Tailwind for styling, React for interactive components (search).

## Tech Stack

- Astro 5
- Tailwind CSS 4
- React 19
- TypeScript
- Satori + @resvg/resvg-js for OG image generation
- Marked for rendering project README markdown

## Key Architecture

- Content collections: `blog` and `portfolio` in `src/content/` using `glob()` loader
- Custom slug system via `src/utils/slugify.ts` — uses `postSlug`/`projectSlug` frontmatter fields with title fallback (does NOT use Astro's built-in `.slug`)
- OG images: `src/pages/[ogTitle].svg.ts` + `src/utils/generateOgImage.tsx`
- Color system: CSS variables as `rgb(r g b)` values mapped to Tailwind via `@theme inline` in `src/styles/base.css`
- Scoped `<style>` blocks in `.astro` files require `@reference "../styles/base.css"` to use `@apply`

## Completed Upgrades

All major upgrades done (committed in `bf5543e`):

1. **Astro 3 → 5** — Content collections use `glob()` loader from `astro/loaders`. `render(entry)` instead of `entry.render()`. Endpoints return `new Response(...)` and export `GET` (not `get`).

2. **React 18 → 19**

3. **Tailwind 3 → 4** — `@tailwindcss/vite` plugin in `astro.config.mjs`. No `tailwind.config.cjs`. CSS-first config in `src/styles/base.css` with `@import "tailwindcss"`, `@theme inline` for color mappings. Opacity modifiers use `/alpha` syntax (e.g. `bg-skin-accent/70`). Tailwind 4 preflight sets `outline-style: solid` globally — all `outline-*` utilities must be under `focus-visible:`.

4. **Cleanup** — fuse.js v7, remark-toc/remark-collapse updated, site URL set to `https://kalmanodds.com`, Node.js engine set to `>=22`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kalmanodds)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kalmanodds)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
