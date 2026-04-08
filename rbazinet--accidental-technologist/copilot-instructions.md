## accidental-technologist

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll-based blog site called "Accidental Technologist" focused on entrepreneurship, technology, and software development. The site uses Jekyll 4.4.1 with the Minima theme and includes features like pagination, search, SEO optimization, and Google Analytics.

## Common Development Commands

### Local Development
```bash
# Start development server (auto-reloads on changes, warnings suppressed)
./serve.sh

# Alternative: Start development server with warnings
bundle exec jekyll serve

# Start with livereload and no warnings
SASS_SILENCE_DEPRECATIONS=import,color-functions,global-builtin bundle exec jekyll serve --livereload

# Build the site
bundle exec jekyll build

# Create a new post with jekyll-compose
bundle exec jekyll post "Post Title"

# Create a new draft
bundle exec jekyll draft "Draft Title"

# Publish a draft
bundle exec jekyll publish _drafts/draft-title.md

# Create a new page
bundle exec jekyll page "Page Title"
```

### Dependency Management
```bash
# Install dependencies
bundle install

# Update dependencies
bundle update
```

## Architecture & Structure

### Core Jekyll Configuration
- `_config.yml`: Main configuration file with site settings, SEO defaults, and plugin configuration
- Uses pagination (10 posts per page) with URL pattern `/page:num/`
- Configured plugins: jekyll-feed, jekyll-seo-tag, jekyll-analytics, jekyll-compose, jekyll-paginate, jekyll-sitemap

### Content Structure
- `_posts/`: Blog posts in Markdown format (YYYY-MM-DD-title.md naming convention)
- `_layouts/`: Custom layout templates (currently has default.html)
- `_includes/`: Reusable HTML components (footer.html, header.html, social.html)
- `assets/`: Static assets including images, CSS, and JavaScript
- `_site/`: Generated static site output (git-ignored)

### Key Features
- **Search functionality**: Uses Simple Jekyll Search with data generated in `search.json`
- **SEO optimization**: Default meta tags and OpenGraph settings configured for all posts and pages
- **Google Analytics**: Integrated via jekyll-analytics plugin with tracking ID G-V8GPSLGQH0
- **Pagination**: Custom pagination implementation in index.html with contextual page numbers

### Important Notes
- The site currently has merge conflicts in several post files (shown in git status)
- Ruby version is managed via `.ruby-version` file
- Development uses bundler to manage Ruby gem dependencies
- The default post layout includes author information and SEO metadata

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rbazinet)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rbazinet)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
