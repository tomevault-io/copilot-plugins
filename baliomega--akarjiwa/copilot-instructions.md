## akarjiwa

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Akarjiwa is a static HTML/CSS/JavaScript website from 2009 that serves as a personal love story website. It's a vintage web portfolio featuring:

- **Main landing page** (`index.html`) with image-based navigation
- **Interactive content pages** opened in lightbox overlays
- **Personal story sections**: "Jiwa Berkata" (tabbed interface) and "Cerita Cinta" (story layout)
- **Visual galleries**: Static and animated visual content sections

## Architecture

### Core Structure
```
/
├── index.html          # Main landing page with navigation
├── jiwa-berkata.html   # Tabbed personal stories (Izwan & Halily)
├── cerita-cinta.html   # Love story narrative
├── visual-statik.html  # Static visual gallery
├── visual-gerak.html   # Animated visual gallery
├── css/
│   ├── style.css       # Main stylesheet with layout and positioning
│   ├── reset.css       # CSS reset
│   ├── lightview.css   # Lightbox modal styles
│   └── tabs.css        # Tab interface styles
├── js/
│   ├── lightview.js    # Lightbox functionality
│   ├── fabtabulous.js  # Tab switching functionality
│   └── tabs.js         # Additional tab utilities
└── images/             # Background images and graphics
```

### Technology Stack
- **HTML**: XHTML 1.0 Strict DOCTYPE
- **CSS**: Traditional CSS with image-based design
- **JavaScript**: Prototype.js framework with Scriptaculous effects
- **External CDNs**: Google-hosted Prototype and Scriptaculous libraries

### Key Patterns
- **Image-based navigation**: Links positioned absolutely over background images
- **Lightbox modals**: Content pages opened as iframe overlays using Lightview
- **Tab interface**: "Jiwa Berkata" uses custom tab switching with Scriptaculous effects
- **Absolute positioning**: Heavy use of absolute positioning for design layout

## Development Notes

### Styling Approach
- Design is image-heavy with precise absolute positioning
- Text content uses Georgia serif font with specific color scheme
- Layout is fixed-width (989px) centered design
- Background images define the visual structure

### JavaScript Dependencies
- Requires Prototype.js (1.6.0.3) and Scriptaculous (1.8.1) from Google CDN
- Custom tab switching with fade/blind effects
- Lightbox functionality for modal content display

### File Encoding
- Uses ISO-8859-1 character encoding
- Contains Malay language content

This is a legacy website representing early 2000s web design patterns and should be treated as a historical artifact when making modifications.

---
> Source: [baliomega/akarjiwa](https://github.com/baliomega/akarjiwa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
