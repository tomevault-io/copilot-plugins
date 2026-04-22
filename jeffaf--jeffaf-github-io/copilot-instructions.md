## jeffaf-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Hugo Site Management
```bash
# Start local development server with drafts
hugo server -D

# Start development server (published content only)
hugo server

# Build site for production
hugo

# Create new post
hugo new posts/my-new-post.md

# Create new page
hugo new about/index.md

# Check Hugo version
hugo version
```

### Content Management
```bash
# View site structure
hugo list all

# Check for broken links or issues
hugo --verbose

# Clean public directory
rm -rf public/
```

## Site Architecture

This is a Hugo static site blog for Jeff Barron, focused on offensive security and malware development topics. The site uses the minimalist **Risotto** theme with a terminal-inspired aesthetic.

### Key Configuration (`hugo.toml`)
- **Base URL**: `https://jeffaf.github.io/`
- **Theme**: Risotto (terminal-inspired minimalist theme)
- **Language**: English (en-us)
- **Color Palette**: `base16-dark` (terminal theme)
- **Author**: Jeff Barron
- **Description**: "Offensive security professional and malware developer"

### Content Structure

#### `/content/`
- **`_index.md`**: Homepage content
- **`about/index.md`**: About page with professional focus and contact info
- **`posts/`**: Blog posts directory
- **`projects/index.md`**: Projects showcase page

#### Navigation Menu
1. **Home** (`/`)
2. **Posts** (`/posts/`) - Blog posts
3. **About** (`/about/`) - Professional background
4. **Projects** (`/projects/`) - Project portfolio

### Theme Details (Risotto)
- **Base16 color schemes** with 16 built-in palettes
- **CSS Grid** for responsive layout without breakpoints
- **No JavaScript** - pure HTML/CSS
- **Terminal aesthetic** - inspired by terminal ricing
- **Current palette**: `base16-dark`

### Social Links Configuration
- **GitHub**: `https://github.com/jeffaf`
- **Twitter**: `https://twitter.com/_jeffaf`
- **Email**: `jeffbarron@protonmail.com`

### Technical Features
- **Taxonomies**: Categories, tags, series
- **Table of Contents**: Levels 2-3, ordered
- **Goldmark renderer**: Unsafe HTML enabled
- **SEO**: robots.txt enabled, social meta tags
- **Pagination**: 3 posts per page

## Content Creation Guidelines

### New Blog Posts
```bash
# Create new post with proper frontmatter
hugo new posts/post-title.md
```

Frontmatter structure:
```yaml
---
title: "Post Title"
date: 2024-01-01
draft: false
tags: ["tag1", "tag2"]
categories: ["category"]
---
```

### Content Focus Areas
Based on the site description and existing content:
- Offensive security techniques and tools
- Malware development and analysis
- Security research and findings
- Project showcases (like bazzy development)
- Learning experiences with Nim and maldev

### File Organization
- Use descriptive filenames with hyphens
- Place images in `static/` directory
- Use relative links for internal content
- Leverage Hugo's built-in shortcodes when possible

## Deployment

This site is configured for GitHub Pages deployment at `jeffaf.github.io`. The `public/` directory contains the built site files.

### Build Process
1. Run `hugo` to generate static files in `public/`
2. Commit and push changes to trigger GitHub Pages deployment
3. Site automatically deploys from the repository

## Theme Customization

The Risotto theme supports extensive customization through:
- **Color palettes**: Switch via `params.theme.palette` in `hugo.toml`
- **Custom CSS**: Add files to `static/css/` directory
- **Favicon**: Place favicon files directly in `static/` directory

Available palettes: `apprentice`, `base16-dark`, `base16-light`, `dracula`, `gruvbox-dark`, `gruvbox-light`, `material`, `papercolor-dark`, `papercolor-light`, `solarized-dark`, `solarized-light`, `tender`, `tokyo-night-dark`, `tokyo-night-light`, `windows-95`, `windows-95-light`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jeffaf) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
