## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

This is a static portfolio website for Cody Lampman, a Growth Product Designer. The site is built with vanilla HTML and CSS, designed to be hosted on GitHub Pages.

**Key files:**
- `index.html` - Main portfolio page with hero section, work samples, experience, about section, and reviews
- `styles.css` - Complete styling with mobile-first responsive design
- `CNAME` - Domain configuration for GitHub Pages (codylampman.com)
- `Cody-Lampman-Resume.pdf` - Downloadable resume
- `images/` - Directory containing all site assets

## Architecture

This is a single-page application with no build process or dependencies. The site uses:

- **Pure HTML5** with semantic structure
- **CSS3** with custom properties (CSS variables) for theming
- **Mobile-first responsive design** with breakpoints at 875px
- **Smooth scroll navigation** to page sections
- **CSS-only hamburger menu** for mobile navigation

## Design System

The site uses a minimal design system defined in CSS custom properties:

```css
:root {
    --background: #F7FFFF;
    --color-primary: #BFD8DE;
    --color-secondary: #EF4444;
    --color-text: #313738;
    --color-subhead: #68868C;
    --font-sans-serif: -apple-system, BlinkMacSystemFont, ...;
    --font-serif: Iowan Old Style, Apple Garamond, ...;
}
```

## Common Development Tasks

Since this is a static site with no build process, development is straightforward:

1. **Local development** - Open `index.html` directly in a browser or use a simple HTTP server
2. **Testing changes** - Refresh the browser after editing HTML/CSS files
3. **Deployment** - Push changes to the main branch; GitHub Pages automatically deploys

## Content Structure

The page is organized into semantic sections:
- Hero section with testimonial quote and introduction
- Work section featuring MailerLite case study
- Experience section with recent roles
- About section with skills and background
- Reviews section with client testimonials
- Contact CTA section

## Styling Approach

- Mobile-first CSS with progressive enhancement
- Semantic HTML structure with accessible navigation
- CSS Grid and Flexbox for layouts
- Custom CSS resets following modern best practices
- Hover states and transitions for interactive elements

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/codylampman)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/codylampman)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
