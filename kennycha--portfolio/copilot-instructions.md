## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev` - Start development server with hot reload
- `npm run build` - Build for production (TypeScript compilation + Vite build)
- `npm run lint` - Run ESLint checks
- `npm run preview` - Preview production build locally

## Architecture Overview

This is a React + TypeScript portfolio application built with Vite that dynamically generates landing pages from TOML configuration files. The app serves as both a portfolio showcase and a landing page generator.

### Core Architecture

**TOML-Based Configuration System**: Each project in `/public/{project-name}/config.toml` defines a complete landing page structure including theme, hero section, features, testimonials, and footer. The app parses these TOML files to generate responsive landing pages.

**Component Structure**:
- `PurePreview` - Core landing page renderer that takes parsed TOML config
- `LandingPreview` - Wrapper that handles TOML parsing and passes to PurePreview
- `ProjectSelector` - UI for switching between different project configurations
- Section components (`HeroSection`, `FeaturesSection`, etc.) - Modular landing page sections

**Routing**: Uses React Router with hash routing (`createHashRouter`) for GitHub Pages compatibility. Three main routes: Projects (/), Careers (/careers), About (/about).

**State Management**: Project configurations are stored in `/public/{project-name}/` directories with assets and config.toml. The `PROJECT_NAMES` constant in `constants.ts` defines available projects.

### Key Files

- `src/lib/toml.ts` - TOML parsing logic with snake_case to camelCase conversion
- `src/lib/constants.ts` - Project definitions and default TOML template
- `src/lib/googleFonts.ts` - Font loading utilities
- `vite.config.ts` - Configured with `/portfolio/` base path for deployment

### Styling

Uses Tailwind CSS v4 with Radix UI components. Theme customization is handled through parsed TOML config with support for custom colors and Google Fonts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kennycha) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
