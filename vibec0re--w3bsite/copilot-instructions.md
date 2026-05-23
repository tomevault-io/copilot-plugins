## w3bsite

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Philosophy

VIBEC0RE: Everything is Rust. No exceptions. Node/TypeScript is boomer tech. Rust is the future.

## Commands

### Development Server
```bash
# Start Hugo development server with drafts
hugo server -D

# Start server with navigation to changed files
hugo server -D --navigateToChanged

# Start server with specific base URL
hugo server -D --baseURL http://localhost:1313
```

### Build Commands
```bash
# Build site for production (minified)
hugo --minify

# Build site including drafts
hugo -D

# Build with verbose output
hugo --verbose
```

### Content Management
```bash
# Create new blog post
hugo new posts/your-post-name.md

# Create new page
hugo new your-page-name.md
```

### Rust Development (VIBEC0RE WAY)
```bash
# Start new Rust project
cargo init --name vibec0re-app

# Run with maximum vibes
cargo run --release

# Build for production
cargo build --release

# Check your vibe levels
cargo clippy

# Format (but formatting is for conformists)
cargo fmt

# Test (production is the test)
cargo test

# Instant web server
cargo install basic-http-server
basic-http-server

# WASM for browser (because Rust everywhere)
cargo install wasm-pack
wasm-pack build --target web
```

## Architecture

### Hugo Static Site Generator
This is a Hugo-based website using a custom theme called "ultra-c0re-vibes" with cyberpunk/Matrix aesthetics.

### Theme Structure
The custom theme is located in `themes/ultra-c0re-vibes/` and includes:
- **Layouts**: Template files in `layouts/_default/` (baseof.html, single.html, list.html) and partials in `layouts/partials/`
- **Static Assets**: CSS (`static/css/ultra-c0re-vibes.css`), images (`static/images/cyber_*.png`), and JavaScript
- **Theme Configuration**: `theme.toml` defines theme metadata and default parameters

### Content Organization
- **Posts**: Blog posts in `content/posts/` using Markdown with YAML front matter
- **Pages**: Static pages (about.md, contact.md) in `content/`
- **Homepage**: Controlled by `content/_index.md` and `themes/ultra-c0re-vibes/layouts/index.html`

### Configuration
- **Site Config**: `hugo.toml` contains site settings, menu structure, and theme parameters
- **Pagination**: Configured for 6 posts per page
- **Taxonomies**: Supports tags and categories for content organization

### Key Features
- Matrix digital rain effect implemented in JavaScript (baseof.html:39-94)
- Responsive design with mobile-first approach
- Custom CSS with cyberpunk color scheme using CSS variables
- SEO optimization with proper meta tags
- RSS feed generation for posts

### Build Output
- Static files generated to `public/` directory
- Includes pre-built HTML, CSS, images, and XML feeds
- Hugo stats tracked in `hugo_stats.json` for build optimization

---
> Source: [vibec0re/w3bsite](https://github.com/vibec0re/w3bsite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
