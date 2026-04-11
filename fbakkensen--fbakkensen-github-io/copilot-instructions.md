## fbakkensen-github-io

> bundle exec jekyll serve --livereload --future

# AGENTS.md - Jekyll Blog Development Guide

## Build Commands

### First-time Setup
```bash
bundle install
```

### Local Development
```bash
# Start Jekyll server with live reload
bundle exec jekyll serve --livereload --future
# Runs at http://127.0.0.1:4000/
```

### Testing
No unit test framework. Use Playwright MCP for browser testing:
- Navigate with `mcp_playwright_browser_navigate`
- Check console errors with `mcp_playwright_browser_console_messages`
- Test responsive design with `mcp_playwright_browser_resize`
- Take screenshots with `mcp_playwright_browser_take_screenshot`

Test these scenarios:
1. Home page with pagination
2. Individual blog posts with TOC
3. Tags page filtering
4. Navigation flows
5. Responsive layouts (mobile/tablet/desktop)
6. Console errors and network failures

## Project Architecture

### Theme: Terminal/PowerShell UI
- Dark blue backgrounds (#012456), cyan highlights (#00ffff)
- Navigation styled as PowerShell commands (e.g., `PS C:\ALProjects\BCBlog>`)
- Font: Cascadia Code for code, system-ui for body text
- All interactive elements maintain PowerShell metaphor

### Layout Hierarchy
- `base.html` - Master layout (nav, footer, analytics, Mermaid.js)
- `home.html` - Blog listing with sidebar
- `post.html` - Individual post with TOC
- `page.html` - Static pages

### File Structure
- `_posts/`: Blog posts (YYYY-MM-DD-title.md)
- `_layouts/`: Jekyll templates
- `_includes/`: Reusable components (toc.html, nav.html, header.html, google-analytics.html)
- `_sass/terminal-theme.scss`: Theme styles
- `assets/css/main.scss`: Compiled CSS
- `assets/js/`: JavaScript functionality
- `assets/images/YYYY-MM-DD-post-slug/`: Post images

## Code Style Guidelines

### Liquid Templates (Jekyll)
- Use `{% raw %}{%- -%}{% endraw %}` for whitespace control
- Semantic HTML5 elements
- Cache-busting for JS: `?v={% raw %}{{ site.time | date: '%s' }}{% endraw %}`
- Reference assets with `| relative_url` filter

### SCSS Styles
- Use theme variables from `terminal-theme.scss` (ps-bg, ps-cyan, ps-yellow, ps-green, ps-red, ps-magenta)
- BEM-like naming convention
- Mobile-first responsive design with breakpoints at 768px
- Code font mixin: `@include code-font`

### JavaScript
- ES6+ syntax with proper event listeners
- Use `DOMContentLoaded` for initialization
- Modern event handling with addEventListener
- No jQuery - vanilla JS preferred
- Scripts in `assets/js/` with cache-busting in base.html

### Blog Posts (Markdown)
- Format: Kramdown parser with GFM input
- Front matter structure:
```yaml
---
layout: post
title: "Post Title"
date: YYYY-MM-DD
categories: [Category1, Category2]
author: Flemming Bakkensen
description: "SEO-friendly description"
tags: [tag1, tag2]
---
```
- Use `<!--more-->` separator for excerpt
- Add emojis to headers: `## 🚀 Introduction`
- Code blocks with Rouge syntax highlighting
- Include tables, diagrams (Mermaid.js available)
- Target audience: AL developers for Business Central

### Naming Conventions
- Posts: `YYYY-MM-DD-title-slug.md`
- Layouts: kebab-case (base.html, post.html)
- SCSS classes: BEM-like with theme prefixes
- Images: timestamp-based naming in post-specific folders

### Error Handling
- Check for YAML front matter syntax errors
- Validate HTML/CSS in generated `_site/` directory
- Test locally before pushing to GitHub Pages
- Monitor GitHub Actions workflow for build failures

## Configuration

### `_config.yml` Key Settings
- Pagination: 5 posts per page
- Plugins: jekyll-feed, jekyll-seo-tag, jekyll-paginate, jekyll-sitemap
- Markdown: Kramdown with Rouge syntax highlighting
- Google Analytics: G-5FQ1BD5CNH (production only)
- Social links: LinkedIn, GitHub, X/Twitter, Bluesky

### Tags System
- Tags defined in post front matter: `tags: [tag1, tag2]`
- `/tags.html` uses JavaScript for tag cloud and filtering
- No manual tag pages needed - auto-generated

## Common Tasks

### Adding a New Blog Post
1. Create `_posts/YYYY-MM-DD-title.md`
2. Add front matter with required fields
3. Write content with `<!--more-->` separator
4. Add emojis to headers
5. Include code examples with syntax highlighting
6. Test locally with `bundle exec jekyll serve --livereload`

### Modifying the Theme
1. Colors/Fonts: Edit `_sass/terminal-theme.scss`
2. Layout structure: Modify `_layouts/base.html`
3. Navigation: Update `_includes/nav.html`
4. Post template: Edit `_layouts/post.html`

### Build Process
- Jekyll builds to `_site/` directory (git-ignored)
- GitHub Pages auto-deploys from main branch
- Nightly rebuild via `.github/workflows/nightly-build.yml`

## Troubleshooting

### Common Issues
- Server won't start: Run `bundle install`
- Changes not reflecting: Check YAML front matter syntax
- TOC not appearing: Ensure h2-h6 headers exist
- Pagination broken: Verify `paginate: 5` in config

### Debugging
- Check Jekyll build output for errors
- Use browser DevTools for console errors
- Validate generated HTML in `_site/` directory

## Do Not Modify

- `_site/` directory (Jekyll build output)
- `Gemfile.lock` (unless updating dependencies)
- `.github/instructions/playwright.instructions.md` (testing rules)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FBakkensen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/FBakkensen)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
