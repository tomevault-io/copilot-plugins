## aldunchev-portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal portfolio website built with Next.js 15, featuring a blog, MDX support, and SEO optimization. The site is deployed on Vercel and uses Tailwind CSS v4 for styling.

## Development Commands

- `npm run dev` - Start the development server
- `npm run build` - Build the production application
- `npm start` - Start the production server

Note: The project supports both npm and pnpm (pnpm-lock.yaml is present).

## Architecture

### App Router Structure

The project uses Next.js App Router with the following key areas:

- **Root Layout** (`app/layout.tsx`): Global layout with Navbar, Footer, Chatbot, and analytics integrations
- **Blog System** (`app/blog/`): MDX-based blog with dynamic routing
- **Static Pages**: `about-me/`, `chatbot/`
- **API Routes**: `og/`, `rss/`, `sitemap.ts`, `robots.ts`

### Blog Architecture

The blog system is file-based using MDX files stored in `app/blog/posts/`:

- **Posts**: MDX files with YAML frontmatter containing `title`, `publishedAt`, `summary`, and optional `image`
- **Utilities** (`app/blog/utils.ts`): Core functions for parsing MDX frontmatter and reading blog posts
  - `getBlogPosts()` - Returns all blog posts with metadata, slug, and content
  - `formatDate()` - Formats dates with relative time
- **Dynamic Routes** (`app/blog/[slug]/page.tsx`): Generates static pages for each blog post
- **Rendering** (`app/components/mdx.tsx`): Custom MDX components with syntax highlighting via `sugar-high`

### MDX Components

Custom MDX components are defined in `app/components/mdx.tsx`:
- Auto-generated heading IDs with anchor links
- Syntax highlighted code blocks (using sugar-high)
- Custom Link component (internal vs external detection)
- Rounded images
- Table support

### Styling System

- **Tailwind CSS v4** with PostCSS integration
- **Custom container utility**: Max width defined in CSS custom property `--container` (670px)
- **Global styles** (`app/global.css`): Includes prose styles, syntax highlighting colors, custom animations
- **Dark mode**: Automatic via `prefers-color-scheme`
- **Utility helper** (`app/lib/utils.ts`): `cn()` function combining clsx and tailwind-merge

### Key Features

- **SEO**: JSON-LD schema, Open Graph images, Twitter cards, sitemap, robots.txt
- **RSS Feed** (`app/rss/route.ts`): Auto-generated from blog posts
- **Dynamic OG Images** (`app/og/route.tsx`): Generated via `next/og` ImageResponse
- **Analytics**: Vercel Analytics and Speed Insights integrated
- **Fonts**: Geist Sans and Geist Mono
- **Chatbot**: Gradio-based chatbot component with external script loaded in layout

### Styling Conventions

- Use the `.container` class for content containers (max-width: 670px)
- Prose styling is applied via `.prose` class for blog content
- Navigation uses custom `.navigation__link` and `.navigation__link--active` classes
- Section dividers use `.section` and `.section--divider` classes

### Content Management

To add a new blog post:
1. Create a new `.mdx` file in `app/blog/posts/`
2. Add YAML frontmatter with required fields: `title`, `publishedAt`, `summary`
3. The post will automatically appear in the blog list and RSS feed

### Configuration

- **Base URL**: Defined in `app/sitemap.ts` as `https://aldunchev.com`
- **TypeScript**: Strict null checks enabled, but general strict mode disabled
- **Container width**: 670px for optimal readability (defined in CSS custom property)

---
> Source: [aldunchev/aldunchev-portfolio](https://github.com/aldunchev/aldunchev-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
