## flatbread

> Required checkpoints for Flatbread schema-breaking migrations


# Flatbread Schema Migration Checkpoints

- Before changing schema generation, resolver arguments, or shared types, write a before/after contract for IDs, refs, filters, root query names, generated TypeScript, config shape, CLI behavior, and examples.
- Escalate for human approval when changes alter IDs, refs, filters, query names, generated TypeScript, config shape, publish scripts, or example app behavior.
- Validate CLI, codegen, generated GraphQL documents/types, README examples, and `examples/nextjs` for schema/API changes.
- Treat breaking work as a coordinated monorepo release across `flatbread`, `@flatbread/core`, `@flatbread/config`, `@flatbread/codegen`, transformers, and examples.
- Do not publish breaking prereleases as accidental `latest`; document migration notes and rollback before release approval.

---
> Source: [FlatbreadLabs/flatbread](https://github.com/FlatbreadLabs/flatbread) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-11 -->
