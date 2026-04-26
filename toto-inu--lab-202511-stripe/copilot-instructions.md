## lab-202511-stripe

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a lab project for experimenting with Stripe integration. The repository contains a Next.js 16 application using React 19, TypeScript, and Tailwind CSS v4 with Biome for linting and formatting.

## Project Structure

- `next-app/` - Next.js application root
  - `app/` - Next.js App Router directory containing pages and layouts
    - `layout.tsx` - Root layout with Geist font configuration
    - `page.tsx` - Home page component
    - `globals.css` - Global styles with Tailwind and CSS variables for theming
  - `public/` - Static assets
  - Configuration files at root of `next-app/`

## Development Commands

All commands should be run from the `next-app/` directory:

```bash
# Development server (runs on http://localhost:3000)
bun dev

# Production build
bun build

# Start production server
bun start

# Linting (uses Biome)
bun lint

# Formatting (uses Biome)
bun format
```

## Technology Stack

- **Framework**: Next.js 16.0.1 with App Router
- **React**: 19.2.0
- **TypeScript**: ^5
- **Styling**: Tailwind CSS v4 with PostCSS
- **Linting/Formatting**: Biome 2.2.0 (replaces ESLint and Prettier)
- **Package Manager**: Bun (as evidenced by bun.lock)
- **Fonts**: Geist Sans and Geist Mono from next/font/google

## Code Style and Standards

### Biome Configuration

The project uses Biome with the following configuration (next-app/biome.json):

- **Formatting**: 2-space indentation
- **Linting**: Enabled with recommended rules for Next.js and React
- **Import Organization**: Automatic import sorting enabled
- **VCS Integration**: Git-aware with .gitignore respect
- **Excluded Directories**: node_modules, .next, dist, build

### TypeScript Configuration

- Target: ES2017
- Module Resolution: bundler
- Strict mode enabled
- Path alias: `@/*` maps to `./` in next-app directory
- JSX: react-jsx (React 19 automatic JSX runtime)

### Styling Patterns

- Tailwind CSS v4 with inline theme configuration
- CSS variables for theming (--background, --foreground)
- Dark mode support via `prefers-color-scheme`
- Custom fonts configured via CSS variables (--font-geist-sans, --font-geist-mono)

## Key Architectural Notes

1. **App Router Structure**: This project uses Next.js App Router (not Pages Router)
2. **Font Loading**: Google Fonts (Geist family) loaded via next/font/google with variable fonts
3. **Theme System**: CSS variables defined in globals.css, referenced in Tailwind theme
4. **Linting Strategy**: Biome replaces both ESLint and Prettier for unified tooling

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/toto-inu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
