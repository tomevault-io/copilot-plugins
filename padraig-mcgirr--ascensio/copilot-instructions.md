## ascensio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a healthcare dashboard application built with React 19, TypeScript, and Vite. The project uses modern React patterns and includes charting capabilities via Recharts, date utilities from date-fns, and Lucide React for icons. Styling is handled with Tailwind CSS.

## Development Commands

- `npm run dev` - Start development server with hot module replacement
- `npm run build` - Build for production (runs TypeScript compilation then Vite build)
- `npm run lint` - Run ESLint on the codebase
- `npm run preview` - Preview the production build locally

## Architecture

### Build System
- **Vite** for fast development and building
- **TypeScript** with strict configuration split across `tsconfig.app.json` and `tsconfig.node.json`
- **ESLint** for code quality with React-specific rules

### Key Dependencies
- **React 19** with TypeScript support
- **Recharts** for data visualization and charts
- **date-fns** for date manipulation and formatting
- **lucide-react** for icons
- **Tailwind CSS** for styling

### Project Structure
```
src/
├── App.tsx          # Main application component
├── main.tsx         # Application entry point
├── assets/          # Static assets
├── App.css          # Component styles
└── index.css        # Global styles
```

## Development Notes

- The project uses Vite's React plugin for fast refresh during development
- TypeScript configuration is split into separate files for app and Node.js code
- ESLint is configured with React hooks and refresh plugins
- No test runner is currently configured in package.json

---
> Source: [Padraig-McGirr/Ascensio](https://github.com/Padraig-McGirr/Ascensio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
