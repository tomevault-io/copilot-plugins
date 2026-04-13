## agora-ui

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
npm run dev              # Start dev server with Turbopack (localhost:3000)
npm run build            # Production build
npm run start            # Production server

npm run lint             # ESLint check
npm run lint:fix         # ESLint auto-fix
npm run type-check       # TypeScript check
npm run format           # Prettier format all files
npm run format:check     # Prettier check

npm run test             # Unit tests (colocated with source)
npm run test:watch       # Unit tests in watch mode
npm run test:coverage    # Unit tests with coverage report
npm run test:integration # Integration tests (__tests__/integration/)
npm run test:bdd         # Cucumber acceptance tests (__tests__/acceptance/)

# Container Development
docker compose up                    # Start WireMock only (use with npm run dev)
docker compose --profile app up      # Start full containerized stack
docker compose down                  # Stop containers

# Dockerfile Linting (Docker-based, no native installation needed)
make lint-dockerfile     # Run Hadolint on Dockerfile
make security-scan       # Run Checkov security scan
make lint-all            # Run all Dockerfile linting
```

To run a single test file:

```bash
npx jest path/to/file.test.tsx
```

## Architecture

**Stack**: Next.js 16 App Router, React 19, TypeScript 5.x (strict), TailwindCSS 4.x, shadcn/ui

**Component Structure**:

- `components/ui/` - shadcn/ui primitives (Button, etc.)
- `components/features/` - Feature-specific composite components
- `components/layouts/` - Page layout components

**Testing Pyramid**:

- Unit tests (70%): Colocated as `*.test.tsx` next to source files - test internal logic, components, and modules in isolation
- Integration tests (20%): `__tests__/integration/` with Jest + WireMock - test HTTP communication with external backend services only; internal dependencies should be mocked
- Acceptance tests (10%): `__tests__/acceptance/` with Cucumber + Playwright - end-to-end user flows

**State Management Strategy**:

- Server state: React Query (TanStack Query)
- URL state: For shareable/bookmarkable state
- React Context: Cross-cutting UI state (theme, locale)
- Local state: UI-only component state

## Key Conventions

**Server Components First**: All components are Server Components by default. Use `'use client'` directive only for interactive components.

**License Headers**: All source files require GPL-3.0 license header (see `.specify/memory/constitution.md` for template).

**Design Tokens**: Defined in `app/globals.css` via TailwindCSS 4.x `@theme` directive. Use semantic color names (`bg-primary`, `text-muted-foreground`).

**Utility Function**: Use `cn()` from `@/lib/utils` for merging Tailwind classes.

## Node Version

Requires Node.js 24.x. Volta is configured for automatic version switching (`volta.node` in package.json).

## Active Technologies
- TypeScript 5.x (strict mode), Node.js 24.x LTS + GitHub Dependabot (native GitHub service), npm ecosystem (005-dependency-scan)
- N/A (GitHub-managed vulnerability database) (005-dependency-scan)

- TypeScript 5.x (strict mode), Node.js 24.x LTS + Next.js 16, React 19, Docker, Docker Compose, WireMock, GitHub Actions, Hadolint, Checkov (002-container-dev-cicd)
- N/A (no database for this feature) (002-container-dev-cicd)
- TypeScript 5.x (strict mode) + Next.js 16, React 19, next-intl (i18n library), TailwindCSS 4.x (003-i18n)
- localStorage (user preference persistence) (003-i18n)
- TypeScript 5.x (strict mode) + Next.js 16 (App Router), React 19, next-intl, shadcn/ui, React Hook Form, Zod (004-jwt-auth)
- HTTP-only secure cookies (client-side), WireMock (development API emulation) (004-jwt-auth)

## Recent Changes

- 002-container-dev-cicd: Added TypeScript 5.x (strict mode), Node.js 24.x LTS + Next.js 16, React 19, Docker, Docker Compose, WireMock, GitHub Actions, Hadolint, Checkov
- 004-jwt-auth: Added JWT authentication with login page, dashboard, token refresh, logout, SSR support

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/earelin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/earelin)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
