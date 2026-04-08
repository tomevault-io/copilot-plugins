## ohpg-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll-based GitHub Pages portfolio site for OHPG. It showcases four HarmonyOS (鸿蒙) applications that work with Jellyfin media server: FinVideo, FinMusic, FinBook, and FinAdmin.

## Common Commands

### Local Development

**Quick start (recommended):**
```bash
# First-time setup (installs Ruby, Jekyll, and dependencies)
./setup.sh

# Start the local development server
./start.sh
```

**Manual commands:**
```bash
# Install dependencies
bundle install

# Start development server with live reload
bundle exec jekyll serve --watch --livereload

# Start server and open browser automatically
bundle exec jekyll serve --open-url

# Build site (outputs to _site/)
bundle exec jekyll build

# Clean generated files
bundle exec jekyll clean
```

### Jekyll Configuration

Key configuration files:
- `_config.yml` - Jekyll configuration, theme settings (minima), and custom variables
- `Gemfile` - Ruby dependencies (github-pages gem, webrick)

## Architecture Overview

### Data-Driven Content

Site content is driven by YAML data files in `_data/`:

- **`_data/summary.yml`** - Hero section stats and project logo list
- **`_data/projects.yml`** - Detailed project showcase data (FinVideo, FinMusic, FinBook, FinAdmin)

Each project in `projects.yml` includes: metadata (name, GitHub URL, tags), device-specific previews (mobile/tablet/desktop), and feature descriptions.

### Layout Structure

- **`_layouts/default.html`** - Base HTML template with footer
- **`index.md`** - Main page using `default` layout; contains all content (Hero section, projects showcase, and interactive JavaScript)

### Asset Organization

```
assets/
├── css/
│   ├── style.css      # Main custom styles
│   ├── main.css       # Minimal overrides
│   └── aos.css        # Animate On Scroll library
├── js/
│   └── aos.js         # AOS animation library
└── statics/           # Project images and logos
    ├── FinVideo/
    ├── FinMusic/
    ├── FinBook/
    └── FinAdmin/
```

### Key Features

1. **Interactive Background** - Canvas-based particle animation with mouse/touch interaction
2. **Device Preview Tabs** - JavaScript-driven device type switching (mobile/tablet/desktop) for project showcases
3. **Image Carousel** - Indicator-based image switching within each device view
4. **AOS Animations** - Scroll-triggered fade-in animations

### Important Notes

- Uses `minima` theme with custom CSS overrides
- GitHub Pages compatibility via `github-pages` gem
- Local assets preferred (aos.css, aos.js stored locally rather than CDN)
- Images are stored in `assets/statics/` with project-specific subdirectories

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OHPG)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/OHPG)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
