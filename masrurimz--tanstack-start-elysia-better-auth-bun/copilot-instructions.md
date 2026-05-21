## routing

> - Place route files in the shared `app/routes/` directory following TanStack's file-based routing convention

# Routing

## TanStack Route Structure

- Place route files in the shared `app/routes/` directory following TanStack's file-based routing convention
- Use dot notation for nested routes: `feature.subpage.tsx`, not nested directories
- Name index routes as `feature.index.tsx`
- Import feature components, controllers, and services from their feature directories with absolute paths
- Keep all feature-specific code in feature directories despite routes being in the shared routes folder
- Route files should contain minimal logic, delegating to controllers for data handling and components for UI

## TanStack Route Loader Pattern

- Use route loaders to fetch data during navigation
- Delegate data fetching logic to controllers
- Access loader data with `Route.useLoaderData()`
- Always destructure values from loaders (e.g., `const { pokemonPair } = Route.useLoaderData()`)

## Reference Files

- Root route example: [__root.tsx](mdc:apps/web/src/routes/__root.tsx)

---
> Source: [masrurimz/tanstack-start-elysia-better-auth-bun](https://github.com/masrurimz/tanstack-start-elysia-better-auth-bun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
