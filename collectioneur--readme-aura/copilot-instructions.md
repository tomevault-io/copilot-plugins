## documentation

> Documentation standards and decision logging


# Documentation Standards

- **English only**: All documentation, comments, and text content must be in English.
- **Decision log**: After medium/large changes, add an entry to `docs/decisions.md` using ADR-lite format:
  ```
  ## YYYY-MM-DD: Title
  - **Context**: Why was this needed?
  - **Decision**: What was chosen?
  - **Alternatives**: What else was considered?
  - **Avoid**: What should not be done?
  ```
- **Keep AGENTS.md in sync**: When adding new files or modules, update the Project Map in `AGENTS.md`.
- **Architecture docs**: Keep `docs/architecture.md` current when the pipeline or module structure changes.

---
> Source: [collectioneur/readme-aura](https://github.com/collectioneur/readme-aura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
