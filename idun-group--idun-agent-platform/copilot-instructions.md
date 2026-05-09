## schema

> Rules for working on the idun_agent_schema package

# Idun Agent Schema

Read `libs/idun_agent_schema/CLAUDE.md` before making changes. It contains the full model hierarchy, discriminated unions, and conventions.

This is the **source of truth** for all data shapes. Schema changes start here, then propagate to engine and manager.

- No runtime dependencies beyond Pydantic — this package must stay lightweight.
- Most models use camelCase aliases (`ConfigDict(alias_generator=to_camel, populate_by_name=True)`).
- Discriminated unions use the `type` field as discriminator.
- Guardrails V2 and Observability V2 are the current standards (V1 is deprecated).

---
> Source: [Idun-Group/idun-agent-platform](https://github.com/Idun-Group/idun-agent-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
