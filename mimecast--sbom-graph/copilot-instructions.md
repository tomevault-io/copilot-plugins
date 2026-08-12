## subagent-quality

> Subagent model quality requirements — never use fast/cheap models for code generation


# Subagent Quality Policy

When delegating work to subagents via the Task tool:

- **Never** specify `model: "fast"` for code-generating subagents (codegen-alpha, codegen-beta, codegen-gamma, testing-agent, security-agent).
- Always use the default (inherited) model for substantive work.
- The `fast` model may only be used for trivial lookups or file searches via `explore` subagents.
- All failing tests must be investigated and fixed — never dismissed as "pre-existing".

---
> Source: [mimecast/sbom-graph](https://github.com/mimecast/sbom-graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
