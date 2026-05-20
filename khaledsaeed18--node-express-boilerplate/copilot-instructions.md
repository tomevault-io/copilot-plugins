## node-express-boilerplate

> Copilot-specific wrapper for this repository.

# GitHub Copilot Instructions

Copilot-specific wrapper for this repository.

## Primary Source

Use [AGENTS.md](../AGENTS.md) as the canonical instructions for architecture, commands, hard rules, and scaffolding order.

## Copilot-Specific Notes

- Keep edits small and local. Avoid broad refactors unless explicitly requested.
- Follow the layered boundary contract exactly:
  - Routes -> Controllers -> Services -> Repositories -> Prisma.
- When changing request/response behavior, validate route middleware order in `src/routes/` and error flow in `src/controllers/base.controller.ts`.
- For schema changes, run both `pnpm prisma:migrate` and `pnpm prisma:generate`.
- Before finishing code changes, run `pnpm full-check` and the most relevant tests.

## References

- Shared agent guidance: [AGENTS.md](../AGENTS.md)
- Project overview: [README.md](../README.md)
- Contributor workflow: [CONTRIBUTING.md](../CONTRIBUTING.md)

---
> Source: [KhaledSaeed18/node-express-boilerplate](https://github.com/KhaledSaeed18/node-express-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
