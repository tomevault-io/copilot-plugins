## mf-expo-patterns

> mf-expo patterns and conventions


# mf-expo Patterns

- **GraphQL**: `graphqlRequest` from `src/shared/services/graphql-client`; `setGraphQLAuthToken` / `clearGraphQLAuthToken` for auth
- **Toast**: `useToast()` → `success()`, `error()`, `warning()`, `info()` — from `@/src/shared/hooks/use-toast`
- **Screens**: PascalCase folders (e.g. `ProfileScreen/ProfileScreen.tsx`)
- **Core**: `packages/masterfabric-expo-core` — PascalCase for views
- **Style**: Functional components, TypeScript interfaces, Prettier; avoid enums (use maps)
- **State**: Zustand, React Query; minimize useState/useEffect

---
> Source: [masterfabric/masterfabric-project-tracker](https://github.com/masterfabric/masterfabric-project-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
