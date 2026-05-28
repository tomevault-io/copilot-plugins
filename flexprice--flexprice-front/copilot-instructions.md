## flexprice-front

> This file provides guidance to Claude Code when working with the Flexprice frontend repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with the Flexprice frontend repository.

## Project Overview

Flexprice Frontend is a React + TypeScript dashboard for the Flexprice billing platform — providing usage metering, credit management, pricing configuration, and invoicing UIs.

- **Framework**: React 18 + TypeScript
- **Build Tool**: Vite
- **Styling**: Tailwind CSS + Radix UI components
- **State Management**: Zustand + TanStack Query
- **Testing**: Vitest + Testing Library
- **Linting**: ESLint + Prettier

## Quick Start

```bash
# Install dependencies
npm install

# Start dev server (port 3000)
npm run dev

# Build for production
npm run build
```

## Testing

```bash
# Run all tests (watch mode)
npm test

# Run tests once (CI mode)
npx vitest run

# Run a single test file
npx vitest run src/path/to/file.test.tsx

# Run with coverage
npm run test:coverage
```

## Linting & Formatting

```bash
# Run ESLint
npx eslint src/

# Fix ESLint issues
npx eslint src/ --fix

# Format with Prettier
npm run format
```

## Project Structure

```
src/
├── api/             # API client and service modules
├── assets/          # Static assets (images, icons)
├── components/      # Reusable UI components
├── config/          # App configuration (env vars, constants)
├── constants/       # Shared constants
├── context/         # React context providers
├── core/            # Core business logic utilities
├── hooks/           # Custom React hooks
├── layouts/         # Page layout components
├── lib/             # Third-party library wrappers (utils.ts, etc.)
├── models/          # TypeScript types and interfaces
├── pages/           # Route-level page components
├── store/           # Zustand state stores
├── tests/           # Test setup and global test utilities
├── types/           # Shared TypeScript type definitions
└── utils/           # General utility functions
```

## Key Technologies

| Technology | Purpose |
|-----------|---------|
| React Router v7 | Client-side routing |
| TanStack Query | Server state management and data fetching |
| Zustand | Client-side global state |
| Radix UI | Accessible headless UI primitives |
| Tailwind CSS | Utility-first styling |
| Recharts | Data visualization |
| Zod | Schema validation |
| Supabase | Authentication |
| Sentry | Error monitoring |

## Development Conventions

### Component Structure

- UI components go in `src/components/`
- Page-level components (route targets) go in `src/pages/`
- Layout wrappers go in `src/layouts/`
- Custom hooks go in `src/hooks/`

### API Calls

- Use TanStack Query (`useQuery`, `useMutation`) for all server interactions
- API service functions live in `src/api/`
- Axios is the HTTP client

### Styling

- Use Tailwind utility classes; avoid inline styles
- Use `cn()` from `src/lib/utils.ts` (tailwind-merge) to conditionally apply classes
- Follow existing Radix UI + Tailwind component patterns in `src/components/`

### TypeScript

- Prefer explicit types over `any`; use Zod schemas for runtime validation
- Model definitions and shared types live in `src/models/` and `src/types/`

## Testing Conventions

- Tests co-located with source files or in `src/tests/`
- File naming: `*.test.tsx` or `*.spec.tsx`
- Test setup file: `src/tests/setup.ts`
- Use `@testing-library/react` for component tests

## Environment Variables

Copy `.env.example` to `.env` and fill in values. Variables are prefixed with `VITE_` to be exposed to the client.

## Available Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start dev server on port 3000 |
| `npm run build` | TypeScript check + Vite production build |
| `npm run preview` | Preview production build locally |
| `npm test` | Run Vitest in watch mode |
| `npm run test:coverage` | Run tests with V8 coverage report |
| `npm run format` | Format all source files with Prettier |
| `npm run storybook` | Start Storybook on port 6006 |

---
> Source: [flexprice/flexprice-front](https://github.com/flexprice/flexprice-front) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
