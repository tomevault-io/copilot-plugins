## jekyll-theme-judo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Local development:**
```bash
bundle install
bundle exec jekyll serve
```

**Building the site:**
```bash
bundle exec jekyll build
```

## Architecture

This is a Jekyll theme gem inspired by judo.codes styling. It's designed as a **remote theme** for GitHub Pages with a clean, enterprise-style layout.

### Key Components

**Layouts:**
- `_layouts/home.html` - Hero section + modular content sections  
- `_layouts/default.html` - Standard page layout

**Includes:**
- `_includes/section.html` - Reusable section blocks for features/modules/pricing/etc
- `_includes/header.html` - Site navigation
- `_includes/footer.html` - Site footer

**Styling:**
- `_sass/judo/_variables.scss` - Theme tokens (colors, fonts, sizing)
- `_sass/judo/_components.scss` - Component styles
- `_sass/judo/_base.scss` - Base styles

### Content Structure

**Home layout** supports modular sections via front matter:
```yaml
sections:
  - id: features
    title: "Section Title"
    subtitle: "Section Subtitle"
    blocks:
      - heading: "Block Heading"
        text: "Block content..."
```

**Theme Configuration** in `_config.yml`:
- Navigation links via `nav` array
- Social links via `social` array
- Site metadata (title, subtitle, description)

### Theme Customization

- Modify design tokens in `_sass/judo/_variables.scss`
- Override any include by copying to site and editing
- Uses system fonts by default
- Supports Jekyll SEO and sitemap plugins

### Example Site

The `example/` directory contains a complete demo site showing:
- Hero landing page with CTA
- Multiple content sections (features, modules, pricing, etc.)
- Navigation structure
- Remote theme setup

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BlackBeltTechnology) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
