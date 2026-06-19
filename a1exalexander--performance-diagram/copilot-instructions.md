## performance-diagram

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
- `npm run dev` - Start development server with Vite
- `npm run build` - Build for production (TypeScript compilation + Vite build)
- `npm run lint` - Run ESLint with TypeScript extensions
- `npm run preview` - Preview production build locally

### Package Management
- Uses `pnpm` as package manager (pnpm-lock.yaml present)
- Also has yarn.lock for compatibility

## Architecture & Structure

This is a React TypeScript application for creating performance review radar charts. The app allows users to:

1. **Select evaluation categories** from predefined sets (well-being, communication, english, leadership, PM, etc.)
2. **Configure time periods** with custom labels and colors
3. **Input numerical scores** (0-10) for each skill/period combination
4. **Visualize data** as interactive radar charts using Recharts
5. **Export charts** as PNG images using html2canvas

### Key Components

**Main App** (`src/App.tsx`):
- Single-page application with all functionality in one component
- Uses React hooks for state management with localStorage persistence
- Renders radar chart, data input table, and control buttons

**Data Configuration** (`src/data/`):
- `categories.ts` - Predefined skill categories with evaluation criteria
- `colors.ts` - Color palette for chart periods (23 unique colors)

### State Management
- Uses `useLocalStorage` from usehooks-ts for persistence
- Key state: activeCategory, periods, subjects, marks
- Data automatically syncs between chart and input table

### Chart Implementation
- Built with Recharts RadarChart component
- Supports up to 12 time periods with custom colors
- Dynamic legend and data visualization
- Fixed scale (0-10) for all metrics

### Key Dependencies
- **recharts** - Chart visualization library
- **html2canvas** - Chart export functionality
- **usehooks-ts** - React hooks utilities
- **lodash** - Data manipulation utilities
- **date-fns** - Date formatting for default periods
- **downloadjs** - File download utility

### Styling
- CSS Modules (`App.module.css`)
- Responsive table layout for data input
- Fixed chart dimensions (1000x600px)

---
> Source: [a1exalexander/performance-diagram](https://github.com/a1exalexander/performance-diagram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
