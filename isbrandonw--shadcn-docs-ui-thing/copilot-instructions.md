## shadcn-docs-ui-thing

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **shadcn-docs-nuxt** documentation site built with Nuxt 3, TypeScript, and a comprehensive shadcn-vue component system. It uses the modern `/app` directory structure (Nuxt 4 style) and is designed for building documentation websites with rich UI components.

## Development Commands

```bash
# Install dependencies
pnpm install

# Development server (http://localhost:3000)
pnpm dev

# Production build
pnpm build

# Preview production build  
pnpm preview

# Generate static site
pnpm generate
```

## Architecture & Key Technologies

### Core Stack
- **Nuxt 3** with TypeScript
- **shadcn-docs-nuxt** framework for documentation
- **Tailwind CSS** with custom HSL theme system
- **Nuxt Content** for markdown-based content
- **PNPM** package manager

### UI System
- **70+ shadcn-vue components** in `/app/components/Ui/`
- **Radix Vue** headless primitives
- **tailwind-variants** for component variants
- **@nuxtjs/color-mode** for dark/light themes

### Enhanced Features
- **Form handling**: Vee-validate with Yup validation
- **Data visualization**: Unovis charts, TanStack Vue Table
- **Advanced UI**: DataTables, calendars, command palette, context menus

## Directory Structure

### `/app` Directory (Nuxt 4 Structure)
- `/components/Ui/` - Complete shadcn-vue component library
- `/assets/css/` - Tailwind configuration and global styles
- `/composables/` - Vue composables for reusable logic
- `/utils/` - Utility functions for charts, styles, navigation

### `/content` Directory
- Markdown content managed by Nuxt Content
- Supports MDX-style components in markdown

## Configuration Files

- `nuxt.config.ts` - Nuxt modules and build configuration
- `app.config.ts` - Site metadata, theme, navigation configuration  
- `tailwind.config.js` - Extended Tailwind with custom animations
- `ui-thing.config.ts` - Component generation settings
- `.prettierrc` - Import sorting and Tailwind class organization

## Development Patterns

### Component Usage
- Components auto-import from `/app/components/`
- Use `Ui` prefix for shadcn components (e.g., `<UiButton>`)
- All components are TypeScript-first with proper prop types

### Content Management
- Content lives in `/content/` directory as markdown files
- Use `<ContentDoc>` component to render content
- Supports frontmatter for metadata

### Theming
- HSL-based CSS custom properties in `/app/assets/css/app.css`
- Dark/light mode handled automatically via `@nuxtjs/color-mode`
- Theme configuration in `app.config.ts`

### Form Handling
- Use Vee-validate with Yup schemas for validation
- Form components available in `/app/components/Ui/`
- Examples in component library demonstrate patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/isbrandonw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
