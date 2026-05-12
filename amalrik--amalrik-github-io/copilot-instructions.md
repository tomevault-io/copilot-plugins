## amalrik-github-io

> Personal technical blog built with Jekyll and Chirpy theme, hosted on GitHub Pages. Focuses on Ruby, Ruby on Rails, and software design patterns.

# AGENTS.md - CoffeeDuty Blog

## Project Overview
Personal technical blog built with Jekyll and Chirpy theme, hosted on GitHub Pages. Focuses on Ruby, Ruby on Rails, and software design patterns.

## Tech Stack
- **Static Site Generator**: Jekyll
- **Theme**: Chirpy (jekyll-theme-chirpy)
- **Language**: Ruby
- **Hosting**: GitHub Pages
- **Features**: PWA enabled, dark mode by default

## Site Configuration
- **URL**: https://amalrik.github.io
- **Title**: CoffeeDuty
- **Tagline**: "Just where I document some interesting things. Mostly about ruby"
- **Author**: Amalrik Maia
- **Language**: English (en)
- **Theme Mode**: Dark

## Content Structure

### Posts (`_posts/`)
Technical articles in Markdown. Front matter includes:
- `layout: post`
- `title`: Article title
- `date`: Publication date (YYYY-MM-DD)
- `categories`: Array (e.g., [ruby, design-patterns])
- `tags`: Array of relevant tags

**Recent posts:**
- Railway Oriented Programming survival guide (2026-01-23)
- Rails Delegated Types intro (2025-12-06)
- RubyGems, Bundler & Docker (2023-10-27)
- Setting up Rails 6 with RSpec (2019-03-16)
- Page Object Pattern (2019-03-02)
- Hello World (2015-05-14)

### Tabs (`_tabs/`)
- `about.md` - About page
- `tags.md` - Tags archive
- `categories.md` - Categories archive
- `archives.md` - Date archives

### Data Files (`_data/`)
- `contact.yml` - Contact information
- `share.yml` - Social sharing config

## Commands

### Development
```bash
bundle exec jekyll serve
# or
bundle exec jekyll serve --drafts
```

### Build
```bash
bundle exec jekyll build
```

## GitHub Actions
- `jekyll.yml` - Jekyll build workflow
- `pages-deploy.yml` - GitHub Pages deployment

## Key Files
| File | Purpose |
|------|---------|
| `_config.yml` | Main Jekyll configuration |
| `index.html` | Homepage |
| `Gemfile` | Ruby dependencies |
| `.github/workflows/` | CI/CD pipelines |

## Notes
- Comments disabled (provider empty in config)
- Analytics: Google Analytics (UA-163699100-1)
- Pagination: 10 posts per page
- TOC enabled by default in posts

---
> Source: [amalrik/amalrik.github.io](https://github.com/amalrik/amalrik.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
