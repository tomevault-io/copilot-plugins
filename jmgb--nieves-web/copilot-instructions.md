## nieves-web

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **modernized static portfolio website** for Nieves Calvo - Architect, Interior Designer, Artist, Creative, and Explorer. The site has evolved from a WordPress export to a custom, responsive portfolio with advanced mobile functionality.

## Development Commands

### Local Development Server
```bash
# Primary method - Python HTTP server
python3 -m http.server 8000
# Alternative port if 8000 is in use
python3 -m http.server 8001

# Alternative methods
npx serve .              # If Node.js available
php -S localhost:8000    # If PHP available
```

### Testing Mobile Responsiveness
- Use browser developer tools to simulate mobile devices
- Test Instagram-style 3x3 grid layout at viewport width ≤768px
- Verify fullscreen modal functionality with project text display

## Architecture and Code Structure

### High-Level Architecture
The site uses a **hybrid approach** combining modern responsive design with preserved WordPress structure for SEO and content organization:

1. **Single Page Application (SPA)**: All content in `index.html` with JavaScript-driven interactivity
2. **Progressive Enhancement**: Works without JavaScript, enhanced with dynamic features
3. **Mobile-First Responsive**: Custom CSS with Instagram-style mobile layout
4. **SEO Optimized**: Maintains WordPress sitemaps and meta structure
5. **Cloudflare Worker + Local Assets**: `src/index.js` serves files through `env.ASSETS` (configured in `wrangler.toml`)

### Key Technical Implementation


#### Project Visibility Management
- **All 16 projects currently visible** (no hidden projects as of latest update)
- Projects controlled via manual `hidden-project` CSS class, NOT nth-child selectors
- Order defined in `mapa-proyectos.md` - this is the source of truth for project organization


### File Organization
```
portfolio-nieves/
├── index.html              # Main site (1400+ lines, contains all content)
├── mapa-proyectos.md       # PROJECT ORDER SOURCE OF TRUTH
├── assets/
│   ├── images/
│   │   ├── profile/        # Bio images, logos
│   │   └── projects/       # Organized by category
│   ├── css/custom.css      # Additional styles (if needed)
│
├── data/
│   └── projects.json      # Structured project data (may be outdated)
└── sitemap files          # SEO optimization
```

## Project Management System

### Project Ordering and Visibility
**CRITICAL**: Always check `mapa-proyectos.md` before making project visibility changes. This file contains:
- Definitive project order (1-16)
- Collaboration details (ICRAVE, HBA MIAMI, etc.)
- Awards and recognition per project
- Project categorization by type


## Development Patterns and Conventions

### CSS Methodology
- **Progressive Enhancement**: Base styles work on all devices
- **Mobile-First**: Mobile styles defined first, desktop as enhancements
- **Important Override Pattern**: Uses `!important` for mobile grid transformation
- **WordPress Class Preservation**: Maintains original WP classes for SEO


### Image Management
- **Responsive images**: Multiple sizes for different devices
- **Error handling**: Graceful fallbacks for missing images
- **Lazy loading**: Built-in browser lazy loading
- **Optimization**: Pre-optimized via WP Smush

## Common Maintenance Tasks

### Cloudflare Deployment (Current)
1. Ensure `CLOUDFLARE_API_TOKEN` is present in `.env`
2. GitHub Actions auto-deploys on every push to `main` using `.github/workflows/deploy.yml`
3. Keep `CLOUDFLARE_API_TOKEN` configured as a GitHub Actions secret
4. `npm run deploy` remains the manual fallback deploy path
5. Verify Worker URL and spot-check homepage/project images
6. Keep `.assetsignore` updated because `wrangler.toml` serves assets from repo root (`assets = { directory = "." }`)

### Asset Source of Truth
- Serve static files from local repository assets via Worker `env.ASSETS` (not from external GitHub raw URLs)
- Keep project media under `assets/images/projects/...`
- Keep references in `index.html` as relative paths (e.g. `assets/images/...`)
- Pushing to `main` should trigger production deploy via GitHub Actions when secrets are configured correctly

### Adding New Projects
1. Update `mapa-proyectos.md` with new project details
2. Add project section to `index.html` following existing pattern
3. Add images to appropriate `assets/images/projects/` subdirectory
4. Test mobile grid layout and modal functionality

### Hiding/Showing Projects
1. **NEVER use CSS nth-child selectors** - they break after reordering
2. **Always use manual `hidden-project` class**:
   ```css
   .project-section.hidden-project {
       display: none !important;
   }
   ```
3. Update `mapa-proyectos.md` to reflect visibility changes

### Mobile Layout Testing
- Test at exactly 768px breakpoint
- Verify 3x3 grid formation
- Check modal text extraction and display
- Ensure smooth transitions between layouts

## Performance Considerations

### Optimization Status
- **Images**: Pre-optimized and properly sized
- **CSS**: Inline styles for critical path performance
- **JavaScript**: Minimal, progressive enhancement only
- **SEO**: Preserved WordPress structure and sitemaps

### Bundle Sizes
- **Total**: ~100KB (significant improvement from original 65K+ tokens)
- **CSS**: Inline in HTML for performance
- **Images**: Optimized with multiple responsive sizes

### Cloudflare Worker Cache Policy
- **HTML**: `Cache-Control: public, max-age=3600` (1 hour)
- **Assets** (`/assets/*`, css/js/images/fonts): `Cache-Control: public, max-age=86400` (1 day)
- **Do not use `immutable`** while assets keep stable filenames without hash/versioning
- If an urgent visual/content update is needed before TTL expires, purge Cloudflare cache

### Performance Guardrails (Must Keep)
- Keep critical hero image optimized and prioritized (`fetchpriority="high"` and `loading="eager"` only for above-the-fold image)
- Keep below-the-fold project images as `loading="lazy"`
- Do not reintroduce HTML meta no-cache tags (`Cache-Control`, `Pragma`, `Expires`) because they hurt repeat-load performance
- Validate with `node --test tests/performance-smoke.test.mjs` after performance-related changes

## Language and Content

- **Primary Language**: Spanish (`lang="es"`)
- **SEO Optimized**: Meta tags, Open Graph, structured data
- **Accessibility**: WCAG 2.1 AA compliant structure
- **Content**: Professional portfolio showcasing architecture and interior design projects

## Important Notes

- **No Build Process**: Direct HTML/CSS/JS - no compilation required
- **SEO Preserved**: Maintains WordPress-generated sitemaps and structure
- **Mobile-First**: Advanced responsive design with Instagram-style mobile layout
- **Performance**: Optimized for fast loading and smooth interactions
- **Maintainability**: Clear separation between content and presentation logic

## 🔍 Cross-review con Codex (segundo par de ojos)

Tras una feature/cambio **relevante**, lanzar automáticamente `/codex:review --wait` como gate de IA antes del primer commit.

**Posición en el flujo**: tests/lint verdes → `/codex:review --wait` → (aplicar fixes) → `git add/commit/push`. Pre-commit (no pre-push) para que los fixes entren en el mismo commit y la historia git quede limpia.

**Lanzar SÍ**: features nuevas, refactors multi-archivo, cambios en lógica crítica/seguridad/auth, infra/deploy, migraciones DB, o cualquier cambio donde el coste de un bug sea alto.
**Lanzar NO**: typos, comentarios, logging, formateo, cambios de 1 línea o exploración.
**En duda**: lanzar (coste bajo, upside alto).

**Si hay hallazgos serios**: `/codex:rescue --resume "aplica los fixes propuestos"` antes de commit.

**Features grandes con commits incrementales**: una sola review al cerrar la feature con `--scope branch --base main`; los fixes van en un commit final "address codex review" antes del push.

---
> Source: [jmgb/Nieves-Web](https://github.com/jmgb/Nieves-Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
