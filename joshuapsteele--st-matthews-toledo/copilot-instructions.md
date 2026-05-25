## st-matthews-toledo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Hugo-based static website for St. Matthew's Anglican Church in Toledo, Ohio. The site uses Hugo's simple templating system with minimal custom CSS for styling.

## Essential Commands

### Development
- `hugo server` - Start development server with live reload
- `hugo server --drafts` - Include draft content in development

### Build
- `hugo` - Build site for production (outputs to `public/`)
- `hugo --gc --minify` - Production build with garbage collection and minification

### Deployment
The site deploys to Netlify automatically with these commands:
- Production: `hugo --gc --minify`
- Deploy previews: `hugo --gc --minify --buildFuture -b $DEPLOY_PRIME_URL`

## Architecture

### Content Structure
- `content/_index.md` - Homepage content (markdown with front matter)
- All main content is in Markdown files in the `content/` directory
- Front matter uses YAML format with `title`, `date`, and `draft` fields

### Layout Structure
- `layouts/index.html` - Homepage template (very simple, just renders content)
- `layouts/_default/baseof.html` - Base template for all pages
- `layouts/_default/` - Default page templates

### Configuration
- `hugo.toml` - Main Hugo configuration
  - Site URL: `https://stmatthewstoledo.org`
  - Contact info stored in `[params]` section
  - Unsafe HTML rendering enabled for embedded content
- `netlify.toml` - Netlify deployment configuration with Hugo 0.140.2

### Styling
- `static/css/style.css` - Main stylesheet
- CSS is directly linked, no build process required
- Responsive design with mobile-friendly layout

## Important Site Information

### Contact Details (from hugo.toml)
- Phone: (419) 473-1187
- Email: StMatthewsACT@gmail.com
- Address: 5240 Talmadge Rd, Toledo, OH 43623
- Pastor: Father Zeke Coughlin

### External Links
- Facebook: facebook.com/people/St-Matthews-Anglican-Church-Toledo/61579296465664/
- YouTube: youtube.com/@stmatthewsepiscopalchurcht6968
- Google Maps embedded for location

## Development Notes

- Hugo version 0.140.2 is specified in netlify.toml
- Site uses minimal dependencies - just Hugo and CSS
- Content is managed through Markdown files, no CMS
- Static assets (images, CSS) go in `static/` directory
- Built site appears in `public/` directory after running `hugo`

---
> Source: [joshuapsteele/st-matthews-toledo](https://github.com/joshuapsteele/st-matthews-toledo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
