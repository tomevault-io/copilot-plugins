## cvportfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This is a portfolio/CV website built with Next.js 15, TypeScript, and Tailwind CSS. The project uses the App Router with internationalization support for French and English.

## Key Commands

### Development
```bash
npm run dev          # Start development server with Turbopack
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint
npm run format       # Check Prettier formatting
```

### Git Workflow
- Main development branch: `dev`
- Feature branches should be created from `dev`

## Architecture & Structure

### Core Technologies
- **Next.js 15.0.4** with App Router and Turbopack
- **TypeScript** with strict mode
- **Tailwind CSS** for styling with custom theme extensions
- **next-intl** for internationalization (French/English)
- **next-themes** for dark/light mode support
- **Motion** (Framer Motion fork) for animations
- **Radix UI** and custom components following shadcn/ui patterns

### Directory Structure
```
src/
├── app/
│   ├── [locale]/              # Internationalized pages
│   │   ├── homepage-sections/ # Modular homepage components
│   │   ├── layout.tsx         # Root layout with providers
│   │   └── page.tsx           # Homepage
│   ├── data/                  # Static data (experience, technologies)
│   └── fonts/                 # Custom Geist fonts
├── components/
│   ├── ui/                    # Reusable UI components (shadcn/ui pattern)
│   └── [feature components]   # Feature-specific components
├── i18n/                      # Internationalization configuration
└── lib/                       # Utilities (cn, theme utilities)
```

### Key Architectural Patterns

1. **Internationalization**: All routes are wrapped in `[locale]` dynamic segment. Use `useTranslations()` hook from next-intl for translations.

2. **Component Organization**: 
   - UI components in `components/ui/` follow shadcn/ui patterns with variants
   - Homepage sections are modular components in `app/[locale]/homepage-sections/`
   - Components use TypeScript with proper prop typing

3. **Styling Strategy**:
   - Tailwind CSS for utility classes
   - CSS variables for theming (defined in globals.css)
   - Custom animations using Motion library
   - Theme colors follow HSL format for dark/light mode compatibility

4. **State Management**:
   - React Context for feature-specific state (e.g., TechWindowContext)
   - next-themes for theme persistence
   - No global state management library

5. **Performance Optimizations**:
   - React Compiler enabled (experimental)
   - Standalone output mode for optimized deployments
   - Turbopack for faster development builds

### Important Configuration Notes

- **Path Aliases**: Use `@/*` for imports from `src/*`
- **ESLint**: Uses @antfu/eslint-config with less opinionated settings
- **React Compiler**: Enabled in next.config.js (experimental feature)
- **Build Settings**: ESLint warnings don't block builds (`ignoreDuringBuilds: true`)

### Development Guidelines

1. When adding new components, follow the existing shadcn/ui pattern with variants
2. All user-facing text should use the translation system
3. Maintain consistent theming by using CSS variables and Tailwind theme extensions
4. Homepage sections should be self-contained components
5. Use TypeScript strict mode - avoid `any` types

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SenouciMoulay) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
