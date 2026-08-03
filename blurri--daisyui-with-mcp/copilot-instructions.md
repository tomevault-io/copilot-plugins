## daisyui-with-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This is a Next.js 15 application with TypeScript, Tailwind CSS v4, and DaisyUI. It uses the Next.js App Router architecture with Turbopack for improved performance.

## Common Commands

### Development
```bash
npm run dev         # Start development server with Turbopack on http://localhost:3000
```

### Building
```bash
npm run build      # Build the application for production with Turbopack
npm run start      # Start the production server
```

### Code Quality
```bash
npm run lint       # Run ESLint to check for code issues
```

Note: There is no dedicated typecheck script. TypeScript checking occurs during the build process.

### Storybook
```bash
npm run storybook       # Start Storybook dev server on http://localhost:6006
npm run build-storybook # Build static Storybook site
```

## Project Structure

### Core Architecture
- **Next.js App Router**: All pages and layouts are in `src/app/`
- **Styling**: Tailwind CSS v4 with DaisyUI plugin loaded via `@plugin "daisyui"` in `globals.css`
- **TypeScript**: Strict mode enabled with path alias `@/*` mapping to `src/*`
- **Component Development**: Storybook configured with Next.js Vite integration for isolated component development

### Key Files
- `src/app/layout.tsx`: Root layout wrapper for all pages
- `src/app/page.tsx`: Home page component
- `src/app/globals.css`: Global styles with custom DaisyUI business-light and business-dark themes for real estate applications
- `src/components/`: Reusable React components
- `src/stories/`: Storybook stories for component documentation

### Dependencies
- **UI Framework**: DaisyUI v5 for component library
- **Runtime**: Next.js 15.5.2, React 19.1.0
- **Build Tools**: Tailwind CSS v4.1.12 using PostCSS configuration

## Development Notes
- Turbopack is enabled by default for both dev and build commands for faster compilation
- ESLint is configured with Next.js core-web-vitals, TypeScript, and Storybook rules
- The project uses React 19's latest features
- Custom DaisyUI themes are defined directly in `globals.css` using Tailwind CSS v4's `@plugin` syntax
- No separate `tailwind.config.ts` file - configuration is handled through PostCSS

---
> Source: [Blurri/daisyui-with-mcp](https://github.com/Blurri/daisyui-with-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
