## doc-maintenance

> Keep documentation and READMEs accurate and updated


- Whenever you receive a correction or correct a mistake, make sure it's documented in .cursor/rules either by updating an existing doc or adding a new one
- Whenever you make changes to a package, make sure its docs stay up to date.
  - **User-facing packages**: keep `README.md` focused on what the package does, when to use it, and a few concrete examples. Avoid internal implementation detail unless it materially affects usage.
  - **Contributor details**: put responsibilities/dependencies/architecture diagrams/internal exports in `DEVELOPING.md` (or `CONTRIBUTING.md`) within that package, and add a short link from `README.md`.
- Docs must not link to transient project artifacts under `projects/` (link to durable `docs/` or package READMEs instead)
- If you establish a new design decision, update the architecture docs in `docs/architecture docs`

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
