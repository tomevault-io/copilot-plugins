## oyi77-github-io

> - Site metadata in `_config.yml`


# Jekyll Development Rules

## Site Configuration

### _config.yml Structure
- Site metadata in `_config.yml`
- Use YAML front matter for page-specific settings
- Jekyll data files in `_data/` directory

### Data Files
- Terminal data: `_data/terminal.yml`
- Companies data: `_data/companies.yml`
- Navigation: `_data/navigation.yml`
- Projects: `_data/projects.json`

## Page Structure

### Front Matter
All pages in `_pages/` must include front matter:

```yaml
---
layout: terminal
title: Page Title
description: Page description
---
```

### Layouts
- Terminal pages use `layout: terminal`
- Layout files in `_layouts/terminal.html`
- Standard pages use `layout: single`

## Posts

### Post Structure
- Posts in `_posts/` directory
- Filename format: `YYYY-MM-DD-post-title.md`
- Must include front matter with required fields

### Post Front Matter
```yaml
---
layout: single
title: Post Title
date: YYYY-MM-DD HH:MM:SS +0000
categories: [Category]
tags: [tag1, tag2]
---
```

## Data Access

### JavaScript Data Injection
- Jekyll data is injected into `window.JEKYLL_DATA`
- Access pattern: `window.JEKYLL_DATA?.terminal?.sites`
- Always use optional chaining for safety

### YAML Data Structure
- Use consistent YAML structure across data files
- Nested objects for organization
- Arrays for lists

## File Organization

### Directory Structure
```
_pages/          # Static pages
_posts/          # Blog posts
_layouts/        # Layout templates
_data/           # Data files (YAML/JSON)
assets/          # Static assets
  css/           # Stylesheets
  js/            # JavaScript files
  images/        # Images
```

### Asset Paths
- Use `/assets/` prefix for asset references
- Relative paths from site root
- Jekyll processes assets automatically

## Plugins

### Required Plugins
- `jekyll-paginate` - Pagination
- `jekyll-sitemap` - Sitemap generation
- `jekyll-gist` - GitHub Gist support
- `jekyll-feed` - RSS feed
- `jemoji` - Emoji support
- `jekyll-include-cache` - Include caching

### Plugin Configuration
Configure in `_config.yml`:
```yaml
plugins:
  - jekyll-paginate
  - jekyll-sitemap
  # ... other plugins
```

## Build & Deployment

### Local Development
```bash
bundle install
bundle exec jekyll serve
```

### GitHub Pages
- Auto-deploys on push to `main` branch
- Uses `github-pages` gem
- No build step required (GitHub builds automatically)

## Best Practices

1. **Use data files** for structured content (not hardcoded in templates)
2. **Consistent front matter** across all pages
3. **Organize assets** by type in subdirectories
4. **Use layouts** for reusable page structures
5. **Liquid syntax** for dynamic content
6. **YAML formatting** must be valid (indentation matters)
7. **Test locally** before pushing to GitHub Pages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oyi77)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/oyi77)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
