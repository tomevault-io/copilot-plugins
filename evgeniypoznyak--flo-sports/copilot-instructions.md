## flo-sports

> Nx monorepo conventions for NestJS backend, Angular frontend, and shared libraries


# Nx Monorepo Rules (NestJS + Angular)

## Global Nx Monorepo Principles

- This is an Nx integrated monorepo with separate apps and libs. Use Nx concepts like projects, targets (serve, test, lint, build), and generators when suggesting changes.
- Do not create new top-level folders outside `apps` and `libs` without being explicitly asked.
- When adding new code, prefer generating Nx projects or libs (e.g., `nx g @nx/angular:app`, `nx g @nx/nest:app`, `nx g @nx/js:lib`) rather than hand-writing boilerplate.
- Respect existing library boundaries: import only from a lib's public API (`public-api.ts` or `index.ts`) and never from deep internal paths.
- When you need details about projects, dependencies, or generators, rely on Nx metadata (e.g., project graph) rather than guessing structure.

## Angular Frontend (`apps/web`)

- Use Angular best practices: standalone components (if already used), dependency injection, and strong typing for inputs/outputs.
- Do not directly import NestJS or backend-specific modules into Angular code. Use HTTP calls or shared DTO/type libraries under `libs/shared` instead.
- When adding new Angular features, first check if a relevant shared or feature library already exists under `libs`. Prefer placing reusable UI or business logic in libs rather than directly under `apps/web`.
- When adding new routes or components, update the Angular routing configuration and follow existing lazy-loading patterns.
- Keep environment-specific URLs in environment files, not hard-coded in components.

## NestJS Backend (`apps/api`)

- Use standard NestJS patterns: modules, controllers, providers, and DTOs with validation where appropriate.
- Do not depend on Angular or frontend-specific code from the backend. Only use shared libs under `libs/shared` for cross-cutting types or logic.
- When defining HTTP contracts (DTOs, interfaces, enums), place them in a shared lib (e.g., `libs/shared/models`) so both frontend and backend can import from the same source.
- Reuse existing NestJS modules and providers when extending functionality instead of creating parallel, duplicated modules.
- Keep NestJS controllers and modules small and focused, and wire them up through the main `AppModule` or appropriate feature modules.

## Shared Libraries (`libs/shared/*`)

- Shared libs must be **frontend-backend neutral**. Avoid importing Angular, NestJS, or other framework-specific dependencies.
- Use shared libs for models, DTOs, validation schemas, utility functions, and feature-agnostic business logic.
- Ensure the public API (e.g., `src/index.ts` or `src/public-api.ts`) re-exports everything intended for consumers. Only import from this public API.
- Update both Angular and NestJS usages together when making breaking changes to shared types in the same commit when possible.
- Keep shared libs small and cohesive. If a lib grows too large, split it into multiple focused libs.

## Testing & Tooling

- Use Nx commands for running tests and builds (e.g., `nx test <project>`, `nx build <project>`) rather than invoking underlying tools directly.
- Keep tests close to the code they exercise and follow the existing testing framework (Jest, Vitest, etc.) already configured.
- When adding or modifying Nx targets in `project.json` or `nx.json`, preserve existing conventions and naming (build, serve, lint, test).
- Align new scripts in `package.json` with Nx targets instead of bypassing Nx.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/evgeniypoznyak) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
