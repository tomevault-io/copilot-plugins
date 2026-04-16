## kiernan-cloud-landing

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
- `pnpm run dev` - Start Vite development server
- `pnpm run build` - Build for production
- `pnpm run preview` - Preview production build locally

## Architecture

This is a React + Vite landing page for Kiernan Cloud Lab with the following key characteristics:

### Core Structure
- **Single Page Application**: Main content is in `src/App.jsx` with modular components
- **Theme System**: Dark/light theme support via `ThemeProvider` with localStorage persistence
- **Component Architecture**: Reusable UI components in `src/components/` following atomic design principles

### Key Technologies
- **React 18** with JSX
- **Vite** as build tool and dev server
- **Tailwind CSS** with custom theme extension and CSS variables
- **Radix UI** components for accessible UI primitives
- **Framer Motion** for animations
- **React Helmet** for SEO metadata management

### Build Configuration
- **Vite Config**: Standard React + Vite setup with path aliases
- **Path Aliases**: `@/` resolves to `src/`

### Styling System
- **Tailwind**: Configured with custom theme extending shadcn/ui design system
- **CSS Variables**: Theme colors defined as HSL CSS custom properties
- **Dark Mode**: Class-based dark mode implementation
- **Animations**: Custom keyframes for accordion and other UI elements

### SEO and Metadata
- **React Helmet**: SEO meta tag management for the landing page

The codebase emphasizes clean component separation, theme consistency, and modern React development patterns.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kiernan-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
