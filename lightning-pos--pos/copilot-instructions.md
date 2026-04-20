## pos

> - src-ui/schema.graphql - Root graphql schema, contains information about all entities, enum and types. Generated from rust backend.


# Graphql Management

File organization
- src-ui/schema.graphql - Root graphql schema, contains information about all entities, enum and types. Generated from rust backend.
- src-ui/**/*.gql - Custom graphql queries, mutation and subscriptions. Usually placed alongside the page.tsx file in which the gql is primarily used. Do not embed graphql queries inside tsx files.


## Codegen
1. We need to generate the `schema.graphql` to let the frontend know about the graphql structure.
2. Any changes to files in the `src-tauri/src/adapters/graphql` folder should regenerate schema using the command `pnpm schema:export` from the project's `root` directory
3. Run codegen for the new graphql schema and custom gql files using the command `pnpm codegen` from the project's `src-ui` directory to generate type definitions at [graphql.ts](mdc:src-ui/lib/graphql/graphql.ts) automatically

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lightning-pos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
