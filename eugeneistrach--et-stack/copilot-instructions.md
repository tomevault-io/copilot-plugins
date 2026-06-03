## 100-core-file-conventions

> File naming and organization conventions


guidelines:
  directories:
    - Use lowercase with dashes (kebab-case)
    - Group by feature or responsibility
    - Keep nesting to a minimum
    - Use consistent names across features

  files:
    components:
      - Name: component-name.tsx
      - Stories: component-name.stories.tsx
      - Tests: component-name.test.tsx

    backend:
      - Domain logic: module-name.server.ts
      - API layer: module-name.api.ts
      - Database: module-name.schema.ts
      - Types: module-name.types.ts

    routes:
      - Index routes: index.tsx
      - Named routes: route-name.tsx
      - Layout routes: _layout.tsx
      - Auth routes: _auth.tsx

examples:
  component: |
    // user-card.tsx
    export const UserCard = () => { ... }

    // user-card.stories.tsx
    export default { component: UserCard }

    // user-card.test.tsx
    describe('UserCard', () => { ... })

  backend: |
    // users.server.ts
    export const createUser = async () => { ... }

    // users.api.ts
    export const $createUser = createServerFn()

    // users.schema.ts
    export const UsersTable = sqliteTable(...)

key_points:
  naming:
    - Use kebab-case for files and directories
    - Be descriptive and consistent
    - Follow established patterns
    - Use appropriate suffixes

  organization:
    - Group related files together
    - Maintain consistent structure
    - Keep paths predictable
    - Use clear file purposes

---
> Source: [EugenEistrach/et-stack](https://github.com/EugenEistrach/et-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
