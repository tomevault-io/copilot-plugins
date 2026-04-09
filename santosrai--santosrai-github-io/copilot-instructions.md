## santosrai-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev` - Start development server (Next.js) on http://localhost:3000
- `npm run build` - Build the project for production (static export)
- `npm run start` - Start production server
- `npm run lint` - Run ESLint to check code quality

## Architecture Overview

This is a personal portfolio website built with Next.js 15, TypeScript, and Tailwind CSS, configured for static export to GitHub Pages.

### Content Management
The site has migrated from WordPress to local MDX files:
- Blog posts are stored as Markdown files in `_posts/` directory
- Posts contain frontmatter (title, date, excerpt, author) and markdown content
- The `lib/mdx.ts` module handles reading, parsing with gray-matter, and converting to HTML with marked
- Post data structure maintains WordPress-like interface for backward compatibility

### Key Architecture Components

**Static Generation**: Configured for static export (`output: 'export'`) with unoptimized images for GitHub Pages deployment.

**Content Layer**: 
- `lib/mdx.ts` - Core content management functions (getPostBySlug, getAllPosts, getPostsByNumber)
- `lib/types.d.ts` - TypeScript interfaces for Post, FeaturedImage, SearchResults
- `lib/config.ts` - Site configuration (name, URL, social links, branding)

**App Router Structure**:
- `app/layout.tsx` - Root layout with Header/Footer and dark theme configuration
- `app/blog/` - Blog listing and individual post pages
- `app/blog/[slug]/page.tsx` - Dynamic post pages with static generation
- `app/about/`, `app/contact/`, `app/projects/` - Static pages

**Components**:
- `components/Header.tsx`, `components/Footer.tsx` - Layout components
- `components/PostList.tsx` - Displays list of blog posts
- `components/MorePosts.tsx` - Pagination component

### Configuration Details

**Next.js Config**: 
- Static export enabled for GitHub Pages
- SWC minification disabled
- Images unoptimized for static hosting
- Trailing slashes enabled

**Styling**: Tailwind CSS with typography plugin for blog content rendering.

## Content Workflow

When working with blog content:
1. Add new posts as `.md` files in `_posts/` directory
2. Include frontmatter: title, date, excerpt, author
3. Use standard markdown syntax for content
4. The build process automatically converts markdown to HTML and generates static pages

## Development Notes

- Site uses force-static rendering for blog pages
- Metadata and SEO configured through Next.js metadata API
- Dark theme configured globally with color scheme and theme color
- Error handling implemented for missing posts (404 pages)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/santosrai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/santosrai)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
