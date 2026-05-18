## codebase-structure

> Frontend Architecture & Development Guidelines for OpenSource Together


This rule formalizes best practices and principles for working effectively with the OpenSource Together frontend, built with Next.js and a feature-based architecture with hybrid state management (TanStack Query + Zustand).

The goal is to ensure modularity, maintainability, scalability, and an optimal developer experience through clear separation of concerns and consistency in conventions.

🚨 CRITICAL INSTRUCTIONS FOR DEVELOPERS 🚨

You MUST strictly follow these principles and rules. Violations will cause maintenance issues, bugs, and performance degradation.

## Feature-Based Architecture

- Each feature is an autonomous module under src/features/ containing views, components, hooks, services, store, validations, and types.
- Never mix multiple features inside one module.
- Features must not depend directly on each other; favor clear, decoupled APIs.
- The src/app/ folder contains only Next.js routing, with no business logic or UI.

## Code Organization & Structure

src/
├── app/ # Next.js routing only (no business logic or UI)
├── features/ # Autonomous business feature modules
│ └── feature-name/ 
│ ├── components/ # Stateless UI components
│ ├── views/ # Feature-specific views/pages
│ ├── forms/ # Feature-specific forms (React Hook Form components)
│ ├── hooks/ # Business logic hooks
│ ├── services/ # API communication and side-effects
│ ├── stores/ # Zustand stores (UI/client state)
│ ├── validations/ # Zod schemas for runtime validation
│ └── types/ # Feature-specific TypeScript types (.type.ts files)
├── shared/ # Reusable components, hooks, lib, and stores (cross-feature)
│ ├── components/
│ ├── hooks/
│ ├── lib/
│ ├── stores/
│ └── types/ # Generic technical types (.type.ts files, e.g., Nullable<T>)
├── config/ # Global app configuration (e.g., TanStack Query)
├── styles/ # Global styles and Tailwind setup
├── types/ # Global app-level types (e.g., User, ApiResponse, MiddlewareMeta)
└── middleware.ts # Next.js middleware (e.g., auth routing logic)

## Hybrid State Management

- TanStack Query manages server state only (data fetching, caching, API sync).
- Zustand manages client state only (UI state, user preferences).
- Never duplicate data between server and client state.
- Business logic hooks in each feature must use TanStack Query or Zustand based on the state nature.

## Naming Conventions and File Structure

- Files use kebab-case.
- Precise suffixes per file role:
- .component.tsx — UI components
- .view.tsx — main pages/views exported by the feature
- .form.tsx — React Hook Form components with logic
- .hook.ts — business logic hooks (logic + state)
- .service.ts — API calls / communication services
- .store.ts — Zustand stores (UI state)
- .schema.ts — Zod schemas for runtime validation
- .type.ts — TypeScript types and interfaces
- .config.ts — configuration files
- TypeScript exports use PascalCase (interfaces/types) and camelCase for Zod schemas.

## Design & Development Principles

- Modularity: decoupled features, reusable components, custom hooks.
- Predictability: strict typing with TypeScript + Zod, unidirectional state flow.
- Performance: intelligent TanStack Query cache, optimistic updates, minimized re-renders.
- Maintainability: clear responsibility separation, isolated layers, easy testing, integrated debugging tools (TanStack DevTools, Sentry).
- Clear responsibilities:
- Views: UI composition and orchestration.
- Hooks: business logic and state management.
- Services: API calls and communication.
- Components: stateless, reusable UI pieces.

## Best Practices

- Never put business logic or API calls inside UI components.
- Always use business logic hooks to access state and services.
- Validate all incoming data with Zod before usage.
- Centralize TanStack Query configuration in config/query.config.ts.
- Favor full type safety by always defining and importing types from \*.type.ts.
- Never duplicate data between TanStack Query and Zustand — synchronize via events or optimistic updates only.
- Use DevTools (TanStack Query DevTools, Zustand middleware) to debug and monitor state.

## Validation & Testing

- Validate API data with Zod in hooks/services.
- Unit test business logic hooks with mocks.
- Unit test UI components in isolation.
- Each feature must be independently testable and deployable.

## Consequences of Rule Violations

- Increased complexity, hard-to-maintain code.
- Bugs from inconsistent or duplicated state.
- Difficulty scaling and integrating new features.
- Performance issues and poor user experience.
- Ineffective tests and harder debugging.

---
> Source: [opensource-together/opensource-together](https://github.com/opensource-together/opensource-together) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
