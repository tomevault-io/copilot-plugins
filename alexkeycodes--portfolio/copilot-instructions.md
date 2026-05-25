## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal portfolio website built with 11ty (Eleventy), using Nunjucks templating, TailwindCSS with SASS preprocessing, and deployed to Netlify. The site is hosted at https://www.alexanderkey.com.

## Development Commands

### Starting Development
```bash
npm run serve
# Builds CSS and starts Eleventy dev server with live reload at http://localhost:8080
```

### Watching for Changes
```bash
npm run watch
# Runs all watch tasks in parallel: SASS compilation, PostCSS processing, and Eleventy serve
```

### Building for Production
```bash
npm run build
# Compiles CSS (SASS → PostCSS/TailwindCSS) and builds static site to dist/
```

### CSS-Only Commands
```bash
npm run build:sass    # Compile SASS to CSS
npm run build:css     # Compile SASS + run PostCSS (includes TailwindCSS + Autoprefixer)
npm run watch:sass    # Watch SASS files for changes
npm run watch:css     # Watch CSS files for PostCSS processing
```

## Architecture

### Build Pipeline
The project has a two-stage CSS build process:
1. SASS compilation: `src/assets/sass/styles.scss` → `src/assets/css/styles.css`
2. PostCSS processing: Applies TailwindCSS directives and Autoprefixer to the compiled CSS

### Template System
- **Templating Engine**: Nunjucks (`.njk` files)
- **Base Layout**: `src/_includes/layouts/base.njk` - Contains the HTML skeleton with meta tags, favicons, and structured data for SEO
- **Components**: Reusable UI pieces in `src/_includes/components/` (header, footer, homepage sections)
- **Content Extension**: Pages extend `layouts/base.njk` and populate the `{% block content %}` block

### Data Layer
Global data is stored in `src/_data/` and accessible in all templates:
- `client.js`: Client-specific information (name, address, domain, phone)
- `global.js`: Site-wide utilities (e.g., `currentYear` for footer)

Access in templates via `{{ client.name }}` or `{{ global.currentYear }}`

### Eleventy Configuration
Configuration in `.eleventy.js`:
- **Input**: `src/` directory
- **Output**: `dist/` directory
- **Template Formats**: `.njk`, `.md`, `.html`
- **Plugins**:
  - `@quasibit/eleventy-plugin-sitemap`: Auto-generates sitemap.xml
  - `@sherby/eleventy-plugin-files-minifier`: Minifies HTML/CSS/JS for production
  - `@11ty/eleventy-navigation`: Provides navigation helpers
- **Passthrough Copy**: Assets folder copied directly to dist/

### TailwindCSS Configuration
- **Content Sources**: `./src/**/*.njk`, `./src/**/*.html`, `./src/**/*.md`
- **Dark Mode**: Class-based (`class` strategy) - add `dark` class to enable
- Main SASS entry point only imports Tailwind directives (`@tailwind base/components/utilities`)

### Directory Structure
```
src/
├── _data/                  # Global data files (client.js, global.js)
├── _includes/
│   ├── layouts/            # Base layout templates
│   └── components/         # Reusable components (header, footer, homepage/*)
├── assets/
│   ├── css/                # Compiled CSS output
│   ├── sass/               # SASS source files
│   ├── js/                 # JavaScript files (main.js, modal.js)
│   ├── images/             # Image assets
│   └── favicons/           # Favicon files
├── index.njk               # Homepage
├── robots.txt              # Search engine instructions
└── sitemap.xml             # SEO sitemap template

dist/                       # Build output (auto-generated, not in git)
```

## Key Implementation Details

### Adding New Pages
1. Create a `.njk` file in `src/` (e.g., `about.njk`)
2. Add frontmatter with page metadata:
   ```yaml
   ---
   title: Page Title
   description: Page description for SEO
   ---
   ```
3. Extend the base layout and define content:
   ```njk
   {% extends "layouts/base.njk" %}
   {% block content %}
     <!-- Page content here -->
   {% endblock %}
   ```

### Adding New Components
1. Create component file in `src/_includes/components/` (e.g., `my-component.njk`)
2. Include it in pages or layouts: `{% include "components/my-component.njk" %}`

### Styling Guidelines
- Use TailwindCSS utility classes directly in templates
- For custom styles, add them to `src/assets/sass/styles.scss`
- Dark mode: Use Tailwind's `dark:` variant (requires `dark` class on a parent element)

### JavaScript
- Main JavaScript: `src/assets/js/main.js` (included in base layout)
- Modal functionality: `src/assets/js/modal.js` (included in base layout)

## Deployment

The site is configured for Netlify deployment:
- **Build Command**: `npm run build` (specified in `netlify.toml`)
- **Publish Directory**: `dist/`
- **SPA Fallback**: Redirects all routes to `/index.html` for client-side routing

---
> Source: [AlexKeyCodes/portfolio](https://github.com/AlexKeyCodes/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
