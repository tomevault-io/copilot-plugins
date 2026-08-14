## website

> This document provides guidelines for AI agents working on Pedro Campos' personal website, a static site built with HTML and Tailwind CSS.

# AGENTS.md - Agent Guidelines for Website Project

This document provides guidelines for AI agents working on Pedro Campos' personal website, a static site built with HTML and Tailwind CSS.

## Project Overview

A clean, terminal-inspired portfolio website showcasing skills, projects, and experience. Built with HTML, Tailwind CSS, and minimal custom CSS. Features bilingual support (English/Portuguese) with responsive design.

## Build/Lint/Test Commands

### Current Setup
No build system configured - uses Tailwind CLI directly:

```bash
# Install dependencies
npm install

# Development: Watch CSS changes
npx @tailwindcss/cli -i input.css -o output.css --watch

# Production build: Minified CSS
npx @tailwindcss/cli -i input.css -o output.css --minify

# Preview site locally
npx serve .
```

### Recommended Scripts (Add to package.json)
```json
{
  "scripts": {
    "dev": "npx @tailwindcss/cli -i input.css -o output.css --watch",
    "build": "npx @tailwindcss/cli -i input.css -o output.css --minify",
    "preview": "npx serve .",
    "lint:html": "htmlhint **/*.html",
    "lint:css": "stylelint **/*.css --config .stylelintrc.json",
    "format": "prettier --write **/*.{html,css,js,json}"
  }
}
```

### Testing
No tests currently configured. For static site validation:

```bash
# HTML validation (single file)
npx html-validate index.html

# CSS validation
npx stylelint **/*.css

# Accessibility check
npx lighthouse http://localhost:3000 --output html

# Visual regression (future)
# percy snapshot dist/
```

## Code Style Guidelines

### HTML Structure
- Use semantic HTML5 elements (`<header>`, `<section>`, `<main>`, `<footer>`)
- Maintain 2-space indentation
- Lowercase elements/attributes
- Descriptive alt text for images
- BEM-like class naming where appropriate
- Self-closing tags for `<img>`, `<input>`, etc.

### CSS/Tailwind Classes
- Prefer Tailwind utilities over custom CSS
- Group classes: layout → spacing → colors → effects
- Consistent responsive prefixes (`lg:`, `md:`, `sm:`)
- Use theme colors via CSS variables
- Minimal custom CSS in `@theme` block
- Avoid `!important` unless necessary

### Naming Conventions
- **Classes**: kebab-case (`hero-section`, `skill-card`)
- **IDs**: camelCase for anchors (`skills`, `projects`)
- **Files**: lowercase with hyphens (`about-me.html`)
- **Images**: descriptive lowercase (`profile-photo.webp`)
- **CSS Variables**: kebab-case (`--color-primary`)

### Design System
- **Colors**: Background `zinc-800`, Primary `green-500`, Text `white`/`gray-400`
- **Typography**: Fira Code font, weights: `font-semibold`/`font-bold`
- **Spacing**: Tailwind scale (`gap-4`, `mt-4`, `px-4`)
- **Layout**: Max width `1024px`, centered with auto margins

### Layout Patterns
- Flexbox for component layouts (`flex flex-col lg:flex-row`)
- Grid for cards (`grid grid-cols-1 lg:grid-cols-2`)
- Mobile-first responsive design
- Consistent padding/margins
- Terminal-inspired design elements

### Component Patterns
- **Navigation**: `#` prefix links with primary color
- **Links**: `underline decoration-primary hover:decoration-2`
- **Cards**: `border border-gray-400` with consistent padding
- **Lists**: `space-y-1` for vertical spacing
- **Images**: Bordered with `border-b-2 border-primary`

### Responsive Design
- Mobile-first approach with `lg:` overrides
- Hide/show elements (`hidden lg:flex`)
- Touch-friendly interactions
- Flexible layouts for all screen sizes

### Accessibility (WCAG 2.1 AA)
- Semantic markup for screen readers
- Alt text for all images
- Sufficient color contrast (4.5:1 ratio)
- Keyboard navigation support
- Focus indicators for interactive elements

### Error Handling
- Validate HTML before commits
- Test links for 404s
- Check responsive layouts
- Verify color contrast
- Browser compatibility testing

### Imports and Dependencies
- Minimal dependencies (Tailwind CLI + typography plugin)
- CDN fonts (Google Fonts)
- Organize CSS: `@import` fonts first, then Tailwind
- Local assets for critical resources

### File Organization
```
/website/
├── index.html              # Main English page
├── pt/index.html           # Portuguese version
├── input.css               # Tailwind input with theme
├── output.css              # Generated CSS (don't edit)
├── tailwind.config.js      # Tailwind configuration
├── package.json            # Dependencies
└── img/                    # Static assets
```

### Git Workflow
- Frequent commits with descriptive messages
- Conventional commits when possible
- Feature branches for new sections
- Test locally before pushing
- Pull request reviews for major changes

### Performance
- Optimize images (< 500KB each)
- Minimize CSS with purging
- Lazy load large images
- Keep bundle size small
- Use modern formats (WebP)

### Browser Support
- Modern browsers: Chrome, Firefox, Safari, Edge
- Mobile testing required
- Graceful degradation for older browsers
- No IE support

### Internationalization
- Separate HTML files per language
- Shared CSS/assets
- Language URLs (`/pt/` for Portuguese)
- Consistent structure across translations

### Content Guidelines
- Concise, professional content
- American English spelling
- Regular content updates
- Working external links
- Consistent terminal-inspired tone

## AI Agent Instructions

1. **Match Existing Patterns**: Follow established design and code patterns
2. **Validate Changes**: Test HTML/CSS and responsive design
3. **Keep Simple**: Avoid over-engineering for static site
4. **Optimize Assets**: Compress images, minimize CSS
5. **Document Updates**: Update this file for new conventions
6. **Bilingual Awareness**: Maintain consistency across languages
7. **Accessibility First**: Ensure WCAG compliance
8. **Performance Focus**: Optimize for fast loading

---

*Last updated: March 2026*
*Maintained by: Pedro Campos*

---
> Source: [pcampos119104/website](https://github.com/pcampos119104/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
