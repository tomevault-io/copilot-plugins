## eliseofoundation-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a React-based website for Fundación Eliseo (Eliseo Foundation), a psychology/mental health foundation. The project was migrated from static HTML to React using Vite as the build tool.

## Development Commands

```bash
# Start development server with hot reload
npm run dev

# Build for production
npm run build

# Run ESLint
npm run lint

# Format code with Prettier
npm run format

# Preview production build
npm run preview
```

## Architecture

- **Framework**: React 19 with Vite build tool
- **Structure**: Single-page application with component-based sections
- **Routing**: Hash-based navigation using anchor links (no React Router)
- **Styling**: CSS modules per component (Component.css files)

### Component Structure

The app follows a simple linear layout:

```
App.jsx
├── Header.jsx (navigation with anchor links)
├── Inicio.jsx (home/hero section)
├── QuienesSomos.jsx (about us section)
├── Blog.jsx (blog/articles section)
└── Donaciones.jsx (donations section)
```

Each component has its own CSS file for styling. The site uses hash navigation (#INICIO, #QUIENES_SOMOS, #BLOG, #DONACIONES) for section scrolling.

### Assets

- Images are stored in both `/img/` and `/public/img/` directories
- Logo and foundation images are used throughout the site
- Static assets are served from the public directory

## Code Conventions

- Components use default exports
- CSS imports at the top of component files
- Image paths reference `/img/` (served from public directory)
- Spanish language used for section names and navigation
- No TypeScript - plain JavaScript/JSX

## Development Notes

- This is a foundation/non-profit website focused on psychology and mental health
- Site content is in Spanish
- Migration from static HTML to React maintains the original structure and styling
- No external routing library - uses simple hash navigation for sections

---
> Source: [lgemc/Eliseofoundation_Website](https://github.com/lgemc/Eliseofoundation_Website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
