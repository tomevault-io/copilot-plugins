## operation-loki

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev` - Start development server on http://localhost:3000 IMPORTANT: Don't start this automatically, ask me to run it.
- `npm run build` - Build production application
- `npm run start` - Start production server
- `npm run lint` - Run ESLint
- `npm run test` - Run Playwright end-to-end tests

## Architecture

This is a Next.js 15 application using the App Router with:

- **Framework**: Next.js 15 with React 19
- **Styling**: Tailwind CSS 4 with custom CSS variables
- **UI Components**: shadcn/ui components in "new-york" style with Radix primitives
- **Testing**: Playwright for end-to-end testing
- **Fonts**: Geist Sans and Geist Mono from Google Fonts

### Project Structure

- `app/` - Next.js App Router pages and layouts
- `components/ui/` - shadcn/ui components (button, card, input, label, select, textarea)
- `lib/utils.ts` - Utility functions including Tailwind class merging
- `tests/` - Playwright test files
- `public/` - Static assets

### shadcn/ui Configuration

Components are configured with:
- Style: "new-york"
- Base color: neutral
- CSS variables enabled
- Icon library: lucide-react
- Path aliases: `@/components`, `@/lib/utils`, etc.

### Testing

Playwright tests are configured to run against all major browsers (Chrome, Firefox, Safari) with base URL http://localhost:3000. Tests expect the development server to be running manually.

## Important Reminders

- Please don't commit & push without asking!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/John0x) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
