## mywebsite

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio website for Anas Alhalabi, a frontend developer aspiring to become fullstack. The site is a static HTML/CSS/JavaScript portfolio hosted on GitHub Pages at `noiceanas.com`.

## Technology Stack

- **HTML5** - Page structure and semantics
- **CSS3** - Responsive design with Flexbox, gradient backgrounds, backdrop filters
- **Vanilla JavaScript** - DOM manipulation for interactivity
- **WebGL** - Real-time fluid dynamics simulation for interactive cursor effects
- **Hosting** - GitHub Pages with automatic deployment

No build tools, package managers, or dependencies required.

## Project Structure

```
.
├── index.html         # Main portfolio page (English)
├── index-ar.html      # Arabic version of portfolio
├── articles.html      # Articles listing page
├── articles-ar.html   # Arabic articles page
├── styles.css         # All styling (no CSS preprocessing)
├── script.js          # Client-side interactions
├── i18n.js            # Internationalization utilities
├── articles-config.js # Article metadata and configuration
├── galaxy-bg.js       # Three.js galaxy background animation
├── CNAME              # Custom domain configuration (noiceanas.com)
├── components/
│   ├── FluidCursor.js # WebGL fluid dynamics simulation (cursor effect)
│   ├── ProfileCard.js # Profile card component logic
│   └── ProfileCard.css # Profile card styling
├── articles/          # Individual article HTML files
├── assets/            # Images and media files
├── translations/      # Translation JSON files
├── .github/workflows/ # GitHub Actions CI/CD
└── .git/              # Version control
```

## Key Features

- **Hero Section**: Introduction with name, title, description, and developer quote
- **Interactive Fluid Cursor**: WebGL-powered fluid dynamics simulation creating realistic airflow/liquid effects following cursor movement
- **Responsive Design**: Mobile-first approach with breakpoints at 768px and 480px
- **Bilingual Support**: Full English and Arabic versions with i18n support
- **Articles System**: Dynamic article loading with metadata, tags, and read times
- **3D Background**: Three.js galaxy particle system animation
- **Profile Card**: Interactive 3D-transform card with hover effects
- **Social Links**: GitHub and LinkedIn connections with hover effects
- **Mobile Navigation**: Hidden by default, toggles on smaller screens
- **Modern Styling**: Glassmorphism design with backdrop filters and gradient background

## Common Development Tasks

### Testing & Validation

Since this is a static site with no build process, testing involves:

- **Browser Testing**: Open `index.html` directly in a browser or serve locally
- **Local Server**: Run a simple HTTP server to test (e.g., `python -m http.server` or `npx serve`)
- **Responsive Testing**: Use browser DevTools to test at breakpoints (768px, 480px)
- **CSS/JS Changes**: No build step needed—changes are immediately visible

### Deployment

The site automatically deploys to GitHub Pages when code is pushed to the `main` branch via GitHub Actions (see `.github/workflows/`).

Manual deployment is optional since the workflow is automated.

### Code Organization

- **CSS**: All styles are in `styles.css`, organized by component (header, hero, buttons, responsive, etc.)
- **JavaScript**:
  - `script.js` - Main DOM interactions (mobile menu toggle, article loading, smooth scrolling)
  - `components/FluidCursor.js` - WebGL fluid simulation (self-contained, auto-initializing)
  - `components/ProfileCard.js` - Profile card interactions
  - `i18n.js` - Language switching and translation utilities
  - `galaxy-bg.js` - Three.js background animation
  - `articles-config.js` - Article metadata and configuration
- **HTML**: Multi-page layout with semantic markup; separate files for English/Arabic versions
- **Components**: Modular JavaScript components in `/components` directory

## Important Patterns & Guidelines

### Shared Header Component

The header is now a **reusable component** managed by `components/Header.js`. This means:

- **English Header**: Edit `components/header.html` to change the navigation for all English pages
- **Arabic Header**: Edit `components/header-ar.html` to change the navigation for all Arabic pages
- **Auto-injected**: All HTML files automatically load the appropriate header via JavaScript
- **Single Source of Truth**: Update the header component once, and all pages reflect the change instantly

**To add a new navigation link:**

1. Edit the appropriate header file (`components/header.html` or `components/header-ar.html`)
2. Changes apply automatically to all pages without editing individual HTML files

### Responsive Design

- Use `clamp()` for scalable typography (already implemented in hero section)
- Test at: 480px (mobile), 768px (tablet), and desktop widths
- Mobile menu is hidden on desktop, visible on mobile (see `.nav-links` styles)

### Styling

- Modern features in use: CSS gradients, backdrop filters, smooth transitions
- Maintain consistent color scheme: purple gradient background (`#667eea` to `#764ba2`)
- All text uses `rgba(255, 255, 255, ...)` with opacity for layering over gradient

### JavaScript

- No frameworks; focus on vanilla DOM manipulation and WebGL
- Key selectors: `.nav-links`, `.mobile-menu-btn`, `.hero`, `.container`
- Event listeners attach to DOM on `DOMContentLoaded`
- Smooth scrolling handled for anchor links (`href^="#"`)
- **FluidCursor**: Self-contained WebGL fluid simulation class
  - Automatically initializes on page load
  - Creates fixed-position canvas overlay (z-index: 50, pointer-events: none)
  - Uses WebGL shaders for real-time fluid dynamics (Navier-Stokes equations)
  - Configurable parameters: simulation resolution, dissipation, curl, splat force
  - Subtle color palette with low opacity for professional look
- **Three.js**: Used only for galaxy background animation in `galaxy-bg.js`

## Deployment & CI/CD

Two identical workflow files exist in `.github/workflows/`:

- `deploy.yml` and `static.yml` both handle automatic deployment to GitHub Pages
- Triggered on push to `main` branch or manual workflow dispatch
- No environment variables or secrets currently required

## Custom Domain

Site is configured with custom domain `noiceanas.com` via `CNAME` file. Do not modify the CNAME unless changing the domain.

---

Things to remember as todos for now
// Add something called /robot.txt to the root of my website that disallows all web crawlers from accessing any part of the site.
// Add something called /sitemap.xml to the root of my website that lists all the pages on my website for web crawlers.
// Add something called /favicon.ico to the root of my website that is a small icon representing my website.
// Add something called /llm-config.json to the root of my website that contains configuration settings for integrating a language model API.
// Add coderabbit to my repo or qodo

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Noice-Anas)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Noice-Anas)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
