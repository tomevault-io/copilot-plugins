## data-layer-and-repositories

> Data layer conventions with Dexie, repositories, and domain mapping

# Data Layer and Repositories

- IndexedDB via Dexie is initialized in [`src/data/db.ts`](mdc:src/data/db.ts)
  - Database name: `gemini_extension`
  - Tables are versioned; update `version(...).stores({...})` as schemas evolve
- Access data via repositories in [`src/data/repositories`](mdc:src/data/repositories)
  - Example: [`chainPromptRepository`](mdc:src/data/repositories/chainPromptRepository.ts)
  - Responsibilities: ID generation (`nanoid`), timestamps, zod validation, Row↔Domain mapping
- Data sources live under [`src/data/sources`](mdc:src/data/sources)

## Guidelines
- Validate inputs with zod schemas at repository boundaries
- Keep domain types in [`src/domain`](mdc:src/domain)
- Sort/paginate/filter in repositories, not UI components
- Prefer immutable updates and explicit timestamps (`createdAt`, `updatedAt`)

---
> Source: [RonkTsang/gemini-chat-extension](https://github.com/RonkTsang/gemini-chat-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
