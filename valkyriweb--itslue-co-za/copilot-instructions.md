## itslue-co-za

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal blog/portfolio site built with Gatsby.js, using the "Hello Friend" theme. The site is hosted at https://itslue.co.za and features dark mode by default, blog posts about personal thoughts and experiences, and a clean, minimal design.

## Development Commands

- `yarn dev` - Start development server (runs `gatsby clean && gatsby develop`)
- `yarn build` - Build production version (`gatsby build`)
- `yarn sitemap` - Build and serve site locally (`gatsby build && gatsby serve`)
- `yarn format` - Format code with Prettier (`prettier --write "src/**/*.js"`)

## Code Architecture

### Core Structure
- **Gatsby Configuration**: `gatsby-config.js` contains site metadata, plugins, and PostCSS configuration
- **Templates**: Located in `src/templates/` - handles page generation (index, individual pages, tags)
- **Components**: Located in `src/components/` - reusable React components (layout, header, footer, post, etc.)
- **Content**: 
  - Blog posts in `src/posts/` as Markdown files with frontmatter
  - Static pages in `src/pages/` (About, Contact, 404)
  - Images in `src/images/` organized by post/topic

### Key Components
- **Layout** (`src/components/layout.js`): Main wrapper component using GraphQL to fetch site metadata
- **Post** (`src/components/post.js`): Renders individual blog post previews with title, date, excerpt, and cover image
- **Templates** (`src/templates/index.js`): Main page template that queries and displays paginated blog posts

### Styling
- Uses PostCSS with various plugins (nested, mixins, preset-env, cssnano)
- CSS files in `src/styles/` with CSS modules for component-specific styles
- Dark theme by default with light mode toggle capability

### Content Structure
Blog posts require frontmatter with:
- `title` - Post title
- `path` - URL path (e.g., "/post-name")
- `date` - Publication date
- `coverImage` - Path to cover image (optional)
- `author` - Author name
- `excerpt` - Brief description
- `tags` - Array of tags

## Code Quality Tools

- **ESLint**: Configured with Airbnb style guide + Prettier + React Hooks rules
- **Prettier**: Code formatting with 80-character line width, trailing commas
- **Babel**: Configured with Gatsby preset and modern JavaScript features (optional chaining, nullish coalescing)

## Site Configuration

The site metadata in `gatsby-config.js` includes:
- Site title: "it's Lue"
- Default theme: dark
- Posts per page: 5
- Main menu items: About, Contact
- Disqus integration for comments

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/valkyriweb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
