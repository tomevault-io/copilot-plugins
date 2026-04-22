## guigui031-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Install dependencies
pnpm install

# Start development server
pnpm run dev
# Alternative: npm run start

# Build for production
pnpm run build

# Preview production build
pnpm run preview
```

## Architecture Overview

This is an **Astrofy** template - a personal portfolio website built with **Astro 4.x**, **TailwindCSS**, and **DaisyUI**. The site supports blog posts, CV timeline, project showcases, and a store section.

### Key Architecture Points

**Content Collections**: The site uses Astro's content collections system with schemas defined in `src/content/config.ts`:
- `blog` collection: Blog posts with frontmatter (title, description, pubDate, heroImage, badge, tags)
- `store` collection: Store items with pricing, checkout URLs, and custom links

**Layout System**: 
- `BaseLayout.astro`: Main template with sidebar navigation, imports SITE_TITLE/SITE_DESCRIPTION from `src/config.ts`
- `PostLayout.astro`: Specialized layout for blog posts
- `StoreItemLayout.astro`: Layout for store items

**Component Structure**:
- Reusable cards: `Card.astro`, `HorizontalCard.astro`, `HorizontalShopItem.astro`
- Navigation: `SideBar.astro`, `SideBarMenu.astro`, `SideBarFooter.astro`
- CV components: `cv/TimeLine.astro` for timeline entries
- Layout components: `Header.astro`, `Footer.astro`, `BaseHead.astro`

**Routing Pattern**:
- Dynamic routes use `[slug].astro` for individual items
- Paginated routes use `[...page].astro` for lists
- Blog supports tag-based filtering: `blog/tag/[tag]/[...page].astro`

### Site Configuration

Global settings in `src/config.ts`:
- `SITE_TITLE`: Site title (currently "Guillaume Genois")
- `SITE_DESCRIPTION`: Site description  
- `GENERATE_SLUG_FROM_TITLE`: Auto-generate blog slugs from titles
- `TRANSITION_API`: Enable Astro view transitions

Site URL configured in `astro.config.mjs`: `https://guigui031.github.io/`

### Content Management

**Blog Posts**: Markdown files in `src/content/blog/` with required frontmatter fields
**Store Items**: Markdown files in `src/content/store/` with pricing and checkout info
**Static Pages**: Direct `.astro` files in `src/pages/` (index, cv, projects, services, 404)

### Styling System

- **TailwindCSS** with **DaisyUI** component library
- Theme switching via `data-theme` attribute in `BaseLayout.astro`
- Typography plugin included for markdown content
- Path aliases configured: `@components/*`, `@layouts/*`

### Important Notes

- Uses **static site generation** - pagination is incompatible with SSR
- Sidebar active states managed via `sideBarActiveItemID` prop
- RSS feed auto-generated at `/rss.xml`
- Sitemap auto-generated on build

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Guigui031) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
