## hermesai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Package Manager

**ALWAYS USE PNPM** - This project uses pnpm as the package manager. Never use npm or yarn.

## Development Commands

- `pnpm dev` - Start development server (http://localhost:3000)
- `pnpm build` - Build the application for production
- `pnpm start` - Start the production server
- `pnpm lint` - Run ESLint to check for code issues

## Project Architecture

This is a Next.js 15.4.6 application using the App Router pattern with:

- **Framework**: Next.js with TypeScript and Tailwind CSS v4
- **Authentication**: NextAuth.js v5 (beta) for OAuth authentication
  - Google and GitHub providers configured
  - Protected routes via middleware
  - Session management and callbacks
- **Internationalization**: next-intl for multi-language support (en, es, fr, de, it, pt)
- **Structure**: Uses App Router in `/app/[locale]` directory for i18n routing
  - `/app/[locale]/layout.tsx` - Locale-specific layout with Geist fonts
  - `/app/[locale]/page.tsx` - Home page component with translations
  - `/app/[locale]/auth/` - Authentication pages (signin, error)
  - `/app/api/auth/[...nextauth]/` - NextAuth API routes
  - `/app/globals.css` - Global styles
  - `/auth.ts` - NextAuth configuration
  - `/i18n/` - Internationalization configuration
  - `/messages/` - Translation files for each locale
  - `/middleware.ts` - Combined auth + i18n middleware
- **Styling**: Tailwind CSS v4 with PostCSS configuration
- **Path Alias**: `@/*` maps to root directory
- **Fonts**: Geist Sans and Geist Mono fonts via next/font/google

## TypeScript Configuration

- Target: ES2017
- Strict mode enabled
- Uses bundler module resolution
- Next.js plugin configured for enhanced TypeScript support

## ESLint Configuration

Uses Next.js core-web-vitals and TypeScript configurations via FlatCompat for modern ESLint setup.

---
> Source: [pdovhomilja/hermesai](https://github.com/pdovhomilja/hermesai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
