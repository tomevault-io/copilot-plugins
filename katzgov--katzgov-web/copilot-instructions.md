## katzgov-web

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll-based website for Katz Government Affairs, a New Jersey government affairs firm. The site uses the Alembic theme as a remote theme and is hosted on GitHub Pages.

## Architecture

- **Jekyll Site**: Static site generator using Jekyll 4.1+ with Alembic remote theme
- **Content Structure**:
  - `index.md`: Homepage with company introduction
  - `team.md`, `clients.md`, `contact.md`: Main pages
  - `_posts/`: Blog posts directory
  - `blog/index.html`: Blog listing page
- **Theme Customization**:
  - `_config.yml`: Site configuration, navigation, and theme settings
  - `assets/styles.scss`: Custom styles that extend Alembic theme
  - `_includes/site-feature.html`: Custom feature section component
- **Static Assets**: Images, logos, and other media files in root directory

## Development Commands

Since this is a Jekyll site, typical commands would be:

**Setup (when Ruby/Bundler available):**
```bash
bundle install          # Install dependencies
```

**Development:**
```bash
bundle exec jekyll serve # Start development server
bundle exec jekyll build # Build static site
```

**Note**: The development environment may need Ruby and Bundler installed. The site is configured to deploy automatically via GitHub Pages.

## Configuration

- **Site URL**: `https://katzgov.github.io/katzgov-web`
- **Theme**: Alembic remote theme (`daviddarnes/alembic@main`)
- **Key Plugins**: jekyll-sitemap, jekyll-seo-tag, jekyll-feed, jekyll-paginate
- **Navigation**: Defined in `_config.yml` under `navigation_header` and `navigation_footer`

## Content Management

- Pages use front matter with `feature_image` and `feature_text` for hero sections
- Blog posts go in `_posts/` with Jekyll naming convention (YYYY-MM-DD-title.md)
- Custom CSS overrides go in `assets/styles.scss`
- The site uses Jekyll's default layouts from the Alembic theme (page, post, blog)

## Deployment

The site is configured for GitHub Pages deployment with the remote theme approach. Changes to the main branch automatically trigger rebuilds.
- Lets avoid writing html/js/css in the md files. Instead we should put them in _includes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/katzgov)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/katzgov)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
