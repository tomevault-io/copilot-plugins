## web-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js 15.3.3 application using React 19.1.0 with TypeScript. The project uses the App Router architecture and is configured with modern tooling including Biome for linting/formatting and Tailwind CSS 4.x for styling.

## Common Commands

### Development
- `pnpm dev` - Start development server with Turbopack
- `pnpm build` - Build production version
- `pnpm start` - Start production server

### Code Quality
- `pnpm run biome:format` - Format code with Biome
- `pnpm run biome:lint` - Lint code with Biome
- `pnpm run lint` - Run Next.js linting

### Git Hooks
- Pre-commit hook automatically runs `biome:format` and `biome:lint`, staging formatted changes

## Architecture Notes

### Directory Structure
- `src/app/` - Next.js App Router pages and layouts
- `src/components/` - Reusable React components
- `src/lib/` - Utility functions and shared logic
- `src/types/` - TypeScript type definitions
- `src/styles/` - Global styles with Tailwind CSS

### Path Aliases
- `@/*` maps to `./src/*` for cleaner imports

### Styling
- Uses Tailwind CSS 4.x with simplified import syntax: `@import "tailwindcss";`
- PostCSS configured with `@tailwindcss/postcss` plugin
- Global styles in `src/styles/globals.css`

### Code Quality Tools
- **Biome**: Handles both linting and formatting (replaces ESLint + Prettier)
  - 2-space indentation, 120 character line width
  - Import organization enabled
  - Recommended rules enabled
- **TypeScript**: Strict mode enabled
- **Husky**: Git hooks for pre-commit checks

### Language Support
- HTML lang attribute set to "ja" (Japanese)
- Project documentation includes Japanese sections

## Dependencies

### Key Production Dependencies
- Next.js
- React
- React DOM

### Key Development Dependencies
- Biome (linting/formatting)
- Tailwind CSS (styling)
- TypeScript
- Husky (Git hooks)

## Development Practices

- Make frequent, small commits with clear commit messages
- Pre-commit hooks automatically format and lint code, staging changes
- Follow conventional commit format when possible

## Notes

- No testing framework currently configured
- Uses pnpm as package manager
- Renovate bot configured for automated dependency updates
- Clean Git repository with standard commit conventions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tsuno08)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tsuno08)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
