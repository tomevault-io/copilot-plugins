## ailigent-spark

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start development server (runs on port 8080)
npm run build    # Production build
npm run lint     # Run ESLint
npm run preview  # Preview production build
```

## Architecture

This is a React + TypeScript landing page built with Vite and styled with Tailwind CSS + shadcn/ui components.

### Tech Stack
- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite with SWC
- **Styling**: Tailwind CSS with CSS variables for theming
- **UI Components**: shadcn/ui (Radix UI primitives)
- **Routing**: React Router DOM
- **State/Data**: TanStack React Query
- **i18n**: i18next with English and Arabic translations

### Project Structure
- `src/components/` - Page sections (Hero, Header, Footer, Team, etc.) and `ui/` subdirectory for shadcn components
- `src/pages/` - Route components (Index, NotFound)
- `src/hooks/` - Custom hooks (`use-mobile`, `useScrollReveal`, `use-toast`)
- `src/locales/` - Translation files (`en.json`, `ar.json`)
- `src/lib/utils.ts` - Utility functions (cn helper for class merging)

### Key Patterns
- **Path aliases**: Use `@/` prefix (e.g., `@/components`, `@/hooks`, `@/lib`)
- **Dark mode**: Enabled by default via `document.documentElement.classList.add('dark')` in App.tsx
- **Theming**: HSL CSS variables defined in `index.css`, consumed via Tailwind config
- **shadcn/ui**: Add new components with the shadcn CLI; components live in `src/components/ui/`

### Page Sections (Index.tsx)
The main page renders sections in order: Header → Hero → MissionVisionValues → ProductsServices → Projects → Team → Consultation → Footer

---
> Source: [karem505/ailigent-spark](https://github.com/karem505/ailigent-spark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
