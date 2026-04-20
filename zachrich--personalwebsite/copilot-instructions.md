## personalwebsite

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio website with MDX blog system. Built with React 18, TypeScript, Vite, and Tailwind CSS. Uses shadcn/ui components and features a cosmic/space theme.

## Development Commands

```bash
npm run dev              # Start dev server (port 8080)
npm run build            # Build for production (includes RSS/sitemap generation)
npm run lint             # Run ESLint
npm run generate:feed    # Generate RSS feed and sitemap only
npm run preview          # Preview production build
```

## Path Alias

Use `@/` to import from `src/`:
```typescript
import { Button } from "@/components/ui/button";
```

## Blog System

### Writing Posts

Blog posts are MDX files in `src/posts/`. Each post must export a `meta` object:

```typescript
export const meta = {
  title: "Post Title",                    // Required
  date: "2024-12-15T10:00:00.000Z",       // Required: ISO date string
  description: "Post description",         // Required
  tags: ["React", "TypeScript"],          // Required
  draft: false,                           // Optional: hidden in production when true
  readingTime: "5 min read",              // Optional
  coverImage: "/image.jpg",               // Optional
  canonical: "https://example.com"        // Optional
};
```

### Code Blocks

Supports syntax highlighting via highlight.js, with optional filename and line highlighting:

````mdx
```typescript filename="example.ts" highlight={2,4}
// code here
```
````

### Draft Posts

Posts with `draft: true` are visible in development but excluded from production builds, RSS feed, and sitemap.

### RSS/Sitemap

Generated automatically during build. Available at `/feed.xml` and `/sitemap.xml`.

## Architecture

### Key Directories
- `src/components/ui/` - shadcn/ui components
- `src/components/` - Custom components (CodeBlock, MDXComponents, etc.)
- `src/pages/` - Route components (Index, BlogIndex, BlogPost, NotFound)
- `src/posts/` - MDX blog posts
- `src/lib/blog/` - Blog utilities (post loading, RSS generation)
- `src/hooks/` - Custom React hooks

### Blog Post Loading
Posts are loaded via Vite's `import.meta.glob` in `src/lib/blog/posts.ts`. The `loadPosts()` function handles validation, draft filtering, and sorting by date.

### MDX Processing
Configured in `vite.config.ts` with:
- `remark-gfm` - GitHub Flavored Markdown
- `remark-slug` - Heading IDs
- `rehype-highlight` - Syntax highlighting
- `rehype-autolink-headings` - Linked headings

### Styling
- Tailwind CSS with HSL color variables in `src/index.css`
- Dark cosmic theme by default
- Custom animations (shooting stars, floating spaceships)

BASE_SYSTEM_PROMPT = """
You are an expert frontend engineer skilled at crafting beautiful, performant frontend applications.
""""

DISTILLED_AESTHETICS_PROMPT = """
<frontend_aesthetics>
You tend to converge toward generic, "on distribution" outputs. In frontend design, this creates what users call the "AI slop" aesthetic. Avoid this: make creative, distinctive frontends that surprise and delight. Focus on:

Typography: Choose fonts that are beautiful, unique, and interesting. Avoid generic fonts like Arial and Inter; opt instead for distinctive choices that elevate the frontend's aesthetics.

Color & Theme: Commit to a cohesive aesthetic. Use CSS variables for consistency. Dominant colors with sharp accents outperform timid, evenly-distributed palettes. Draw from IDE themes and cultural aesthetics for inspiration.

Motion: Use animations for effects and micro-interactions. Prioritize CSS-only solutions for HTML. Use Motion library for React when available. Focus on high-impact moments: one well-orchestrated page load with staggered reveals (animation-delay) creates more delight than scattered micro-interactions.

Backgrounds: Create atmosphere and depth rather than defaulting to solid colors. Layer CSS gradients, use geometric patterns, or add contextual effects that match the overall aesthetic.

Avoid generic AI-generated aesthetics:
- Overused font families (Inter, Roboto, Arial, system fonts)
- Clichéd color schemes (particularly purple gradients on white backgrounds)
- Predictable layouts and component patterns
- Cookie-cutter design that lacks context-specific character

Interpret creatively and make unexpected choices that feel genuinely designed for the context. Vary between light and dark themes, different fonts, different aesthetics. You still tend to converge on common choices (Space Grotesk, for example) across generations. Avoid this: it is critical that you think outside the box!
</frontend_aesthetics>
"""

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZachRich) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
