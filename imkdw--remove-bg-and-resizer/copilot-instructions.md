## remove-bg-and-resizer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js 15 project for background removal and image resizing functionality. Currently bootstrapped with the default Next.js template but intended to be developed into a tool for image processing tasks.

## Development Commands

### Core Commands
- `pnpm dev` - Start development server with Turbopack (preferred package manager based on lockfile)
- `pnpm build` - Build the application with Turbopack
- `pnpm start` - Start production server
- `pnpm lint` - Run ESLint for code quality checks

### Development Server
The development server runs on http://localhost:3000 with hot reloading enabled.

## Architecture

### Stack
- **Framework**: Next.js 15 with App Router
- **Styling**: Tailwind CSS 4
- **TypeScript**: Fully typed with strict mode enabled
- **Fonts**: Geist Sans and Geist Mono from Vercel
- **Package Manager**: pnpm (evidenced by pnpm-lock.yaml)

### Project Structure
- `src/app/` - Next.js App Router directory structure
  - `layout.tsx` - Root layout with font configuration
  - `page.tsx` - Main page component (currently default Next.js template)
  - `globals.css` - Global styles
- `public/` - Static assets (SVG icons)
- Configuration files at root level

### Configuration
- TypeScript configured with path mapping (`@/*` → `./src/*`)
- ESLint extends Next.js TypeScript rules with FlatCompat for modern config
- PostCSS configured for Tailwind CSS processing
- Next.js config currently minimal (ready for customization)

## Key Implementation Notes

### Current State
The project is currently in initial state with default Next.js template. The main functionality for background removal and resizing needs to be implemented.

### Development Approach
- Uses strict TypeScript configuration
- Path aliases configured for clean imports
- Tailwind CSS for styling with utility-first approach
- App Router structure for modern Next.js development

---
> Source: [imkdw/remove-bg-and-resizer](https://github.com/imkdw/remove-bg-and-resizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
