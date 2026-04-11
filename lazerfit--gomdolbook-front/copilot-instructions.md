## gomdolbook-front

> **gomdolbook** is a reading log and library management web application optimized for PC. It facilitates active reading by encouraging users to answer three key questions about their reading material: "What is the book about?", "Interpret the content", and "Critique it".

# Project Context: gomdolbook-front

## Overview

**gomdolbook** is a reading log and library management web application optimized for PC. It facilitates active reading by encouraging users to answer three key questions about their reading material: "What is the book about?", "Interpret the content", and "Critique it".

### Tech Stack

- **Framework:** React 18 (Vite)
- **Language:** TypeScript
- **Styling:** Styled Components (Global styles in `src/styles/globals.css`)
- **State Management/Data Fetching:** @tanstack/react-query
- **Authentication:** Keycloak (@react-keycloak/web)
- **Testing:** Vitest, React Testing Library, MSW (Mock Service Worker)
- **Routing:** React Router v6

## Architecture & Conventions

### Directory Structure

- `src/api`: API services, configuration (interceptors, endpoints), and types.
- `src/components`: Atomic Design structure (`atoms`, `molecules`, `organisms`, `templates`, `pages`).
- `src/hooks`: Custom React hooks (business logic separation).
- `src/styles`: Theme definitions, global CSS, and mixins.
- `src/test-utils`: Test setup and utilities.

### Coding Standards

- **Path Aliases:** Use `@/` to refer to the `src` directory (configured in `vite.config.mjs` and `tsconfig.json`).
- **Component Pattern:**
  - Functional components with named exports.
  - Props interfaces usually extend HTML attributes (e.g., `React.ButtonHTMLAttributes<HTMLButtonElement>`).
  - Styled components often co-located in the same file.
  - **Transient Props:** Use `$` prefix for styled component props (e.g., `$size`) to prevent them from being passed to the DOM.
- **API Calls:**
  - Endpoints are defined in `src/api/services/config/APIEndpoints.ts` as functions returning URL strings.
  - Services (e.g., `BookService.ts`) use these endpoints.
- **Query Client:** Configured with `staleTime: 1 hour`, `retry: 1`, and `refetchOnWindowFocus: false` (see `src/api/services/config/queryClient.ts`).

### Authentication

- Uses `ReactKeycloakProvider` wrapping the app in `src/index.tsx`.
- Tokens are handled via `onTokens` callback which calls `setToken` (likely updating an Axios interceptor).

## Development Workflow

### Scripts

- **Start Dev Server:** `pnpm dev` (runs `vite`)
- **Build:** `pnpm build` (runs `tsc && vite build`)
- **Test:** `pnpm test` (runs `vitest`)
- **Lint:** `pnpm lint` (runs `eslint`)
- **Format:** `pnpm format` (runs `prettier`)

### Testing

- Tests are co-located with components (e.g., `BookInfo.test.tsx`).
- Uses `jsdom` environment.
- Setup files in `src/test-utils/setupHttp` and `src/setupTests.ts`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lazerfit)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lazerfit)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
