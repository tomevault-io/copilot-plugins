## yourpalette

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static e-commerce website for "Your Palette" - a Korean makeup tools retailer. The site features:
- Homepage with product catalog and shopping cart functionality
- Product detail page with image gallery and specifications
- Bootstrap 4.6-based responsive design
- Vanilla JavaScript for interactivity
- No build process or package manager

## Architecture

The site consists of:
- **index.html** - Main homepage with product grid, hero section, and shopping cart modal
- **product-details.html** - Detailed product page with image gallery, specifications, and related products
- **css/styles.css** - Custom styles using CSS custom properties for theming
- **img/** - Product images and hero background

## Key Features

### Shopping Cart System
- Client-side JavaScript shopping cart with localStorage persistence
- Floating cart icon with item count
- Bootstrap modal for cart display
- Shared cart state between pages

### Product Display
- Responsive product grid using Bootstrap grid system
- Product cards with hover effects
- Image gallery with thumbnail navigation on product detail page
- Related products section

### Styling System
- CSS custom properties for consistent theming:
  - `--tan-primary: #C69C6D` (primary brand color)
  - `--tan-light: #E8DCC6` (light accent)
  - `--tan-dark: #A67C52` (dark accent)
  - `--cream: #F5F5DC` (background)
  - `--charcoal: #2C2C2C` (text)

## Development Notes

### No Build Process
This is a static HTML/CSS/JS website with no build tools, package managers, or preprocessing. All code is browser-ready.

### External Dependencies
- Bootstrap 4.6.2 (CSS & JS) - loaded via CDN
- jQuery 3.6.0 - loaded via CDN
- Font Awesome 6.0.0 - loaded via CDN

### JavaScript Architecture
- Vanilla JavaScript with jQuery for DOM manipulation
- Global state management for shopping cart
- Event handlers for product interactions
- localStorage for cart persistence

### Responsive Design
- Bootstrap grid system for layout
- Custom media queries for mobile optimization
- Mobile-first approach with desktop enhancements

## Common Tasks

Since this is a static site, development typically involves:
- Editing HTML structure directly
- Modifying CSS styles in styles.css
- Updating JavaScript functionality inline in HTML files
- Adding/replacing product images in img/ directory

No testing framework or linting tools are configured.

---
> Source: [tankgriffin/YourPalette](https://github.com/tankgriffin/YourPalette) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
