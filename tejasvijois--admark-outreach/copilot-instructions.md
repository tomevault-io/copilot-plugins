## project

> Core project rules — docs as source of truth, no architecture/schema drift


# Project Rules

## Source of truth

- Treat `docs/` as the source of truth for product, architecture, database, APIs, coding standards, AI behavior, and decisions.
- Before implementing any feature, read the relevant documentation in `docs/` first.
- If documentation conflicts, or a request conflicts with approved docs, stop and explain the conflict. Do not guess or invent a resolution.

## Architecture and schema

- Never redesign or reinterpret approved architecture.
- Never modify the database schema unless explicitly instructed to do so.

## Implementation discipline

- Follow `docs/CODING_STANDARDS.md` for all code changes.
- Implement only the requested task. Do not expand scope, add unrelated refactors, or introduce speculative features.
- Keep changes small, focused, and production-ready.

---
> Source: [TejasviJois/Admark-Outreach](https://github.com/TejasviJois/Admark-Outreach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
