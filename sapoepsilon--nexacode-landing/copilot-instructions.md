## nexacode-landing

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static landing page for a digital services agency (Eazy Technology) specializing in custom websites, mobile apps, and AI integration. The project uses vanilla HTML, CSS, and JavaScript with Tailwind CSS via CDN for styling.

## Development Commands

```bash
# Start local development (opens index.html in browser)
npm start

# Run local development server
npm run serve

# No linting is configured for this static site
npm run lint
```

## Architecture & Structure

This is a single-page application with the following structure:

- **index.html** - Main landing page with all sections (hero, services, portfolio, process, contact)
- **script.js** - Main JavaScript file handling animations, mobile menu, and contact form
- **faq.js** - FAQ accordion functionality
- **Case study pages** - Individual HTML files for portfolio items:
  - estmtagent-case-study.html
  - skilldrop-case-study.html
  - viusbuilt-case-study.html

The site uses Tailwind CSS configured directly in index.html with custom colors:
- Primary: #2D3047 (Deep Indigo)
- Secondary: #1B998B (Vibrant Teal)
- Accent: #FF9B71 (Coral Orange)

### Design System Standards

**Corner Radius Values:**
- Small elements (badges, tags): `rounded` (0.25rem / 4px)
- Medium elements (buttons, inputs): `rounded-lg` (0.5rem / 8px)
- Large elements (cards, containers): `rounded-2xl` (1rem / 16px)
- Extra large elements (hero sections, major containers): `rounded-3xl` (1.5rem / 24px)

Always use these standardized corner radius values for consistency across the site.

## Key Implementation Details

1. **No build process** - This is a static site that runs directly in the browser
2. **Tailwind via CDN** - Custom configuration is embedded in index.html
3. **Contact form** - Currently frontend-only, requires backend integration for functionality
4. **Mobile responsiveness** - Uses Tailwind's responsive utilities throughout
5. **Deployment** - Can be deployed to any static hosting (Netlify configuration included)

## When Making Changes

- All styling should use Tailwind utility classes
- Maintain the existing color scheme unless specifically asked to change it
- The site is optimized for performance - avoid adding heavy dependencies
- Case study pages follow the same structure and styling as the main page

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sapoepsilon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
