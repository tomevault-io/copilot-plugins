## patelpratyush-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a personal portfolio website built with Gatsby v3, React, and styled-components. It's a static site generator that creates pages from React components and markdown content.

## Development Commands

### Setup
```bash
yarn                    # Install dependencies
```

### Development
```bash
npm start               # Start dev server (alias for gatsby develop)
gatsby develop          # Start dev server at http://localhost:8000
gatsby clean            # Clear .cache and public directories (use when build issues occur)
```

### Production
```bash
npm run build           # Create production build in /public
npm run serve           # Preview production build locally
```

### Code Quality
```bash
npm run format          # Format all JS, JSX, JSON, and MD files with Prettier
npx eslint <file>       # Lint specific files (uses @upstatement/eslint-config/react)
```

Note: The project uses husky for pre-commit hooks with lint-staged, which automatically formats and lints staged files.

## Architecture

### Content Management
Content is managed through markdown files in the `/content` directory:
- `/content/jobs/` - Work experience entries
- `/content/featured/` - Featured projects shown prominently on homepage
- `/content/projects/` - Additional project entries
- `/content/posts/` - Blog posts for the "pensieve" section

Markdown files include frontmatter with metadata like title, date, slug, tags, and project-specific fields (company, tech stack, links, etc.).

### Page Generation
- **Static pages**: Defined in `/src/pages/` (index.js, archive.js, 404.js)
- **Dynamic pages**: Generated in `gatsby-node.js` from markdown content
  - Blog posts are created from `/content/posts/` using `src/templates/post.js`
  - Tag pages are created at `/pensieve/tags/{tag}/` using `src/templates/tag.js`

### Component Structure
- `/src/components/` - Reusable UI components (Layout, Nav, Menu, Footer, etc.)
- `/src/components/sections/` - Main homepage sections (Hero, About, Jobs, Featured, Projects, Contact)
- `/src/components/icons/` - SVG icon components

The homepage (index.js) composes these section components in order: Hero → About → Jobs → Featured → Projects → Contact.

### Configuration
- `/src/config.js` - Central configuration for email, social media links, nav links, color scheme, and ScrollReveal settings
- `gatsby-config.js` - Gatsby plugins, site metadata, and markdown processing configuration
- `gatsby-node.js` - Build-time logic including:
  - Webpack aliases (@components, @config, @fonts, @hooks, @images, @pages, @styles, @utils)
  - Dynamic page creation from markdown
  - SSR workarounds for browser-only libraries (scrollreveal, animejs)

### Styling
- Uses styled-components for component styling
- Global styles in `/src/styles/`
- Custom fonts in `/src/fonts/`
- Theme colors defined in config.js (navy, green accent)

### Data Fetching
Gatsby's GraphQL layer queries markdown files transformed by gatsby-transformer-remark. Frontmatter and content are queried in components and templates using GraphQL.

## Key Technical Details

### Import Aliases
Webpack is configured with path aliases for cleaner imports:
```javascript
import { Layout } from '@components';
import config from '@config';
```

### Markdown Processing
gatsby-transformer-remark plugins enhance markdown:
- gatsby-remark-images - Optimized image handling
- gatsby-remark-prismjs - Code syntax highlighting
- gatsby-remark-code-titles - Code block titles
- gatsby-remark-external-links - External link handling

### Animation Libraries
- ScrollReveal - Scroll-based reveal animations (config in srConfig)
- anime.js - Additional animations
- Both are excluded from SSR builds via gatsby-node.js webpack config

### PWA Features
- gatsby-plugin-manifest - Web app manifest with icon
- gatsby-plugin-offline - Service worker for offline support
- gatsby-plugin-sitemap - XML sitemap generation
- gatsby-plugin-robots-txt - Robots.txt generation

## Color Scheme

The site uses a dark navy theme with green accents:
- Navy: #0a192f
- Light Navy: #112240
- Lightest Navy: #233554
- Slate: #8892b0
- Light Slate: #a8b2d1
- Lightest Slate: #ccd6f6
- White: #e6f1ff
- Green (accent): #64ffda

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/patelpratyush)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/patelpratyush)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
