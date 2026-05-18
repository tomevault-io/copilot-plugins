## claraverse

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ClaraVerse OSS is an open-source full-stack application with a React + TypeScript frontend and a Go backend. The project uses Docker Compose for local development with MongoDB, Redis, and SearXNG as supporting services.

## Development Commands

All development commands are run from the `frontend/` directory:

```bash
cd frontend

# Development
npm run dev              # Start Vite dev server on http://localhost:5173

# Building
npm run build            # TypeScript compilation + Vite production build
npm run preview          # Preview production build locally

# Code Quality
npm run lint             # Run ESLint with zero warnings policy (--max-warnings=0)
npm run lint:fix         # Auto-fix ESLint issues
npm run format           # Format code with Prettier
npm run format:check     # Check code formatting without changes
npm run type-check       # Run TypeScript type checking without emitting files
```

## Architecture

### Frontend Structure (React + TypeScript + Vite)

The frontend follows a feature-based architecture with clear separation of concerns:

- **Components**: Split into `ui/` (base components like Button) and `layout/` (structural components like Header, MainLayout)
- **Routing**: Centralized in `src/routes/index.tsx` using React Router v6 with `createBrowserRouter`
- **State Management**: Zustand stores in `src/store/` with devtools and persist middleware
- **API Layer**: Centralized API client in `src/services/api.ts` with typed responses and error handling
- **Path Aliases**: `@/` resolves to `src/` (configured in both vite.config.ts and tsconfig.app.json)

### Key Architectural Patterns

**Store Pattern (Zustand)**:
- Stores use `create()` with `devtools()` and `persist()` middleware
- State and actions defined in single interface
- Example: `useAppStore.ts` manages theme and sidebar state with localStorage persistence

**API Service Pattern**:
- Single API client (`api.ts`) with methods: get, post, put, delete
- Custom `ApiError` class for typed error handling
- Base URL from environment variable: `VITE_API_BASE_URL`

**Route Configuration**:
- Routes wrapped in `MainLayout` component
- Uses `createBrowserRouter` (data router) for better performance than `BrowserRouter`

**Component Organization**:
- Each component directory has `index.ts` for barrel exports
- UI components support variants via props
- Layout components compose the page structure

## Tech Stack

- **React 19** with TypeScript strict mode
- **Vite 7** for build and dev server
- **Tailwind CSS 4** with PostCSS
- **React Router v7** (using v6 patterns)
- **Zustand 5** with devtools and persist middleware
- **ESLint 9** with TypeScript, React Hooks, and Prettier plugins
- **Prettier 3** for code formatting

## Configuration Notes

**TypeScript**:
- Project uses composite configuration (`tsconfig.json` references `tsconfig.app.json` and `tsconfig.node.json`)
- Strict mode enabled with additional rules: `noUnusedLocals`, `noUnusedParameters`, `noFallthroughCasesInSwitch`
- Path alias `@/*` maps to `./src/*`

**ESLint**:
- Flat config format (new ESLint 9 style)
- Enforces Prettier formatting as errors
- Unused vars allowed if prefixed with underscore: `argsIgnorePattern: '^_'`
- Global ignores: `dist`, `node_modules`, `build`

**Environment Variables**:
- Copy `.env.example` to `.env` in frontend directory
- Variables must be prefixed with `VITE_` to be exposed to client code
- Current vars: `VITE_API_BASE_URL`, `VITE_APP_NAME`, `VITE_APP_VERSION`

## Development Workflow

1. Frontend development happens in `frontend/` directory
2. Run `npm run dev` to start development server
3. Before committing: run `npm run format` and ensure `npm run lint` passes
4. Type checking runs during build, but can be checked separately with `npm run type-check`
5. Import components using `@/` prefix for absolute imports (never use relative paths like `../../`)

## Repository Structure

```
├── frontend/          # React + TypeScript frontend (Vite)
├── backend/           # Go backend (API, WebSocket, tools)
├── docs/              # User-facing documentation (FAQ, guides, etc.)
├── scripts/           # Build and install scripts
├── searxng/           # SearXNG search engine config
├── docker-compose.yml # Main Docker Compose for local dev
├── docker-compose.dev.yml
├── .env.example       # Environment template (copy to .env)
└── CLAUDE.md          # This file
```

**Important**: Never commit `.env` files — use `.env.example` as the template.

---
> Source: [claraverse-space/ClaraVerse](https://github.com/claraverse-space/ClaraVerse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
