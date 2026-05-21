## fullstack-auth-template

> - docs/ARCHITECTURE.md

# Full-Stack Authentication Template - Coding Rules

## Reference Documents
- docs/API_SPEC.md
- docs/ARCHITECTURE.md
- docs/DATABASE_SCHEMA.md

## TypeScript Rules
- TypeScript strict mode required
- No `any` type usage
- Explicit type definitions for all functions and variables

## Naming Conventions
- camelCase: variables, functions
- PascalCase: React components, classes, types/interfaces
- UPPER_SNAKE_CASE: constants

## Code Quality
- Error handling required for all async operations
- Use Winston logger instead of console.log
- Zod validation for all API inputs
- React Hook Form for form handling

## File Organization
- Backend: controllers, services, models, routes, middleware
- Frontend: pages, components, hooks, api, utils
- Shared types in validators with Zod schemas















---
> Source: [yamwoong/fullstack-auth-template](https://github.com/yamwoong/fullstack-auth-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
