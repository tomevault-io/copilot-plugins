## astro-md-theme

> **Language Preference: Please communicate in Chinese (中文) when responding to the user.**

# CLAUDE.md

**Language Preference: Please communicate in Chinese (中文) when responding to the user.**

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

This is a **pnpm workspace monorepo** for an Astro-based blog theme:

- **`packages/theme/`**: Reusable theme package (`@chenenpei/astro-md-theme`) containing components, templates, styles, and Tailwind config
- **`apps/theme-demo/`**: Theme demonstration site with example content
- **`apps/my-blog/`**: Personal blog instance using the theme

## Architecture

### Theme Package Pattern

The theme follows a **template-based architecture** where apps consume pre-built page templates from the theme package. The key insight is that apps fetch data using Astro's `getCollection('blog')` and pass it as props to theme templates.

**Example from `apps/theme-demo/src/pages/index.astro`:**
```astro
---
import HomePage from '@chenenpei/astro-md-theme/templates/HomePage.astro';
import { getCollection } from 'astro:content';
import { SITE } from '../config';

const posts = (await getCollection('blog'))
  .sort((a, b) => b.data.pubDate - a.data.pubDate)
  .slice(0, 3)
  .map(post => ({
    title: post.data.title,
    slug: post.slug,
    excerpt: post.data.excerpt,
    date: post.data.pubDate,
    category: post.data.category,
    coverImage: post.data.coverImage,
  }));
---

<HomePage
  siteTitle={SITE.title}
  siteDescription={SITE.description}
  authorName={SITE.author?.name}
  authorBio={SITE.author?.bio}
  authorAvatar={SITE.author?.avatar}
  socials={SITE.socials}
  posts={posts}
/>
```

The theme package exports:
- **Components** (packages/theme/src/components/): Layout, Header, PostCard, IntroCard, PaperSheet, Post
- **Templates** (packages/theme/src/templates/): HomePage, PostDetailPage, ArchivePage, AboutPage
- **Styles**: global.css and Tailwind config

Exports are defined in `packages/theme/package.json` under the `exports` field.

### Content Collections

Blog posts are Markdown files in `apps/{app-name}/src/content/blog/*.md` with frontmatter schema defined in `src/content/config.ts`:

```typescript
{
  title: string,
  excerpt?: string,
  pubDate: Date,
  category?: string,
  readingTime?: string,
  coverImage?: string,
  coverImageAlt?: string
}
```

### Site Configuration

Each app has its own `src/config.ts` exporting a `SITE` object with:
- `title`, `description`
- `author`: { `name`, `bio`, `avatar` }
- `url`
- `socials`: { `x`, `instagram`, `github`, `email` }

This config is used both in `astro.config.mjs` for PWA manifest and passed to theme templates.

### Styling System

- **Global styles**: Imported in `Layout.astro` from theme package
- **Component styles**: Scoped `<style>` blocks in individual `.astro` files
- **Dark mode**: `data-theme` attribute on `<html>`, persisted via localStorage, initialized with inline script in Layout.astro
- **Tailwind**: Extended from theme's `tailwind.config.mjs`, apps use `applyBaseStyles: false` to avoid conflicts

### Markdown Processing

Configured per-app in `astro.config.mjs`:
- **theme-demo**: `remark-math`, `rehype-katex`, `remark-definition-list` (for LaTeX math, definition lists)
- **my-blog**: `@astrojs/mdx` for MDX support

Both use KaTeX CSS (imported in Layout.astro) for math rendering.

## Common Commands

```bash
# Development (starts all apps in parallel)
pnpm dev                    # All apps
pnpm dev:demo              # theme-demo only
pnpm dev:blog              # my-blog only

# Build
pnpm build                  # Build all apps
pnpm --filter <app-name> build  # Build specific app

# Preview production build
pnpm preview

# Dependencies
pnpm install
pnpm update
```

## Workspace Linking

The theme uses `workspace:*` protocol in app dependencies. This means:
- Changes to `packages/theme` hot-reload immediately in running apps
- No manual rebuilding or reinstallation needed
- Theme exports defined in `packages/theme/package.json` (exports field)

## Code Standards

**Code comments**: All code comments MUST be in English.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chenenpei)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chenenpei)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
