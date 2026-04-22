## lola-martin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands
- Build: `npm run build`
- Dev server: `npm run dev`
- Lint: `npx eslint ./src/**/*.{js,jsx,ts,tsx}`
- Type-check: `npx tsc --noEmit`
- Production build: `npm run build && next-sitemap`

## Code Style Guidelines
- **Imports:** Group imports (React, Next.js, external libs, internal)
- **Components:** Use function declarations with explicit prop types
- **Naming:** PascalCase for components, camelCase for functions/variables
- **Styling:** Use Tailwind CSS classes with consistent formatting
- **Error handling:** Use try/catch blocks with appropriate fallbacks
- **File structure:** Group related components in directories
- **State management:** Use hooks (useState, useContext) for state
- **Animation:** Use Framer Motion for animations via MotionComponents
- **Paths:** Use '@/' alias for imports from src directory
- **Internationalization:** Support for Hebrew/English via i18next

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/itayost) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
