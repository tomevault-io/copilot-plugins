## implementation-honesty

> Enforce no fake-success behavior in runtime code and UI.


# Implementation Honesty Rule

- Prototype placeholders are temporary and must be clearly marked.
- Runtime code must not return simulated success for unimplemented behavior.
- Missing integrations/models/providers must return explicit `not_implemented` errors.
- Security/runtime findings must never be synthetic in production paths.
- Webapp must show **Under construction** for unavailable functionality.

---
> Source: [sandraschi/virtualization-mcp](https://github.com/sandraschi/virtualization-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
