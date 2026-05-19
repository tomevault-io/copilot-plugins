## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static portfolio website built with HTML, CSS, and JavaScript using Bootstrap 4.3.1. Based on the "Dream Pulse" template from TemplateMo.

## Running the Website

No build process required. Open `index.html` directly in a browser, or use VS Code's Live Server extension for development with auto-reload.

## Architecture

**Core Files:**
- `index.html` - Single-page site with sections: Intro, About, Work (gallery), Clients, Contact
- `css/templatemo-dream-pulse.css` - Custom styles and responsive breakpoints
- `js/templatemo-scripts.js` - jQuery-based interactions

**Third-Party Libraries (included locally):**
- Bootstrap 4.3.1 (`css/bootstrap.min.css`)
- jQuery (`js/jquery.min.js`)
- Slick Carousel (`slick/`) - Work gallery slider
- Magnific Popup (`js/jquery.magnific-popup.min.js`, `css/magnific-popup.css`) - Image lightbox
- Parallax.js (`js/parallax.min.js`) - Header parallax effect
- SinglePageNav (`js/jquery.singlePageNav.min.js`) - Smooth scroll navigation

**External CDN:**
- Font Awesome 6.2.1 (icons)
- Google Fonts (Open Sans)

## Key Patterns

- Fixed left sidebar with navigation, collapses to hamburger menu below 991px
- Content sections use Bootstrap's grid system
- Gallery uses Slick with responsive breakpoints at 1600px, 1200px, 991px, 767px, 480px
- Image hover effect implemented in CSS as `.effect-bubba`
- Primary color: `#069` (blue)

---
> Source: [magdalena-rutkowska/Portfolio](https://github.com/magdalena-rutkowska/Portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
