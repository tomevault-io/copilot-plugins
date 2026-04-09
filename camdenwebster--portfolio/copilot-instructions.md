## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Core Development
- `npm run dev` - Start Vite development server
- `npm run build` - Build for production
- `npm run lint` - Run ESLint
- `npm run preview` - Preview production build locally

### Blog Management
- `npm run sync-blog` - Sync blog posts from external Obsidian vault
- Set `BLOG_SOURCE_DIR` environment variable to point to your blog source directory

## Architecture Overview

This is a React + TypeScript portfolio website built with Vite, Tailwind CSS, and React Router.

### Key Structure
- **Single Page Application**: Main portfolio content is rendered on the home route (`/`)
- **Blog System**: Separate blog routes at `/blog` and `/blog/:slug` with markdown file support
- **Component Architecture**: Page sections are individual components (Hero, About, Projects, etc.)
- **Theme System**: Dark/light mode toggle with Tailwind CSS dark mode classes
- **Static Blog Content**: Blog posts stored as markdown files in `public/content/blog/`

### Blog Post Management
- Blog posts are markdown files in `/public/content/blog/`
- `src/utils/markdown.ts` contains the `BLOG_POSTS` array mapping slugs to file paths
- The sync script automatically updates this array when copying posts from an external source
- Uses `gray-matter` for frontmatter parsing and `react-markdown` for rendering

### Routing
- `/` - Main portfolio page with all sections
- `/blog` - Blog listing page
- `/blog/:slug` - Individual blog post pages
- Uses React Router with hash-based navigation for sections on the main page

### Styling
- Tailwind CSS with custom color palette
- Dark mode support with `class` strategy
- Typography plugin for markdown content styling
- Custom dark theme colors defined in tailwind config

### Build Configuration
- Vite with React plugin
- Buffer polyfill configured for browser compatibility
- Markdown files included as assets
- Excludes lucide-react from optimization for better performance

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/camdenwebster)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/camdenwebster)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
