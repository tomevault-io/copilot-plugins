## brianbruff-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git instructions
- Never mention Anthropic or Claude in git commit messages

## Commands

### Development
- `npm start` or `npm run develop` - Start development server at http://localhost:8000
- `npm run build` - Build the site for production
- `npm run serve` - Serve the production build locally
- `npm run clean` - Clean Gatsby cache and public directory
- `npm run format` - Format code using Prettier

### Deployment
- The site automatically deploys to GitHub Pages when changes are pushed to the `master` branch via GitHub Actions

## Architecture

This is a Gatsby-based static site generator for Brian Keating's technical blog:

### Key Technologies
- **Gatsby v5** - React-based static site generator
- **React 18** - Component framework
- **GraphQL** - Data layer for querying posts and content
- **Remark/MDX** - Markdown processing for blog posts
- **PrismJS** - Syntax highlighting for code blocks

### Content Structure
- Blog posts are stored as Markdown files in `/src/posts/`
- Posts follow the naming convention: `YYYY-MM-DD-title-slug.md`
- Each post has frontmatter with title, date, and tags
- Images and media assets are stored in `/src/images/` and `/static/`

### Data Flow
1. Markdown posts in `/src/posts/` are processed by `gatsby-transformer-remark`
2. GraphQL nodes are created for each post with slug generation in `gatsby-node.js`
3. Blog post pages are dynamically created at `/blog/[slug]/` using the template at `/src/templates/blog-post.js`
4. The main blog listing page is at `/src/pages/blog.js`

### Dark Theme
The site includes a dark theme with red color scheme. Theme styles are managed through CSS modules and the main layout component.

## Important Notes
- Do what has been asked; nothing more, nothing less
- Never create files unless absolutely necessary for achieving the goal
- Always prefer editing existing files to creating new ones
- Never proactively create documentation files (*.md) or README files unless explicitly requested

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brianbruff)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/brianbruff)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
