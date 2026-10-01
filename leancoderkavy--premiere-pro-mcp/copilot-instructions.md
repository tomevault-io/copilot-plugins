## premiere-pro-mcp

> MCP tool schema, registration, authority, and mutation verification


# MCP tools

Each module exports `getXTools(...)` returning a `Record<string, ToolDef>` with description, JSON Schema `parameters` (every property described), and handler. Register new modules in `src/server.ts` and in `tests/tools/tool-modules.test.ts` when that catalog lists modules.

- Keep schemas, descriptions, registrations, structured results, authority, tests, and `docs/supported-actions.md` synchronized.
- Default authority is `inspect,edit,export,filesystem`. Do not enable `unsafe-script` to paper over a missing operation.
- Mutating tools must read Premiere state back. Report `committed`, `verified`, `committed_unverified`, or failure. A host return value is not proof of success.
- Do not silently retry a failed UXP mutation through CEP or QE.
- Add tests for behavior, failure paths, escaping, validation, and authority. Vitest mocks the bridge; it does not prove a live host.

---
> Source: [leancoderkavy/premiere-pro-mcp](https://github.com/leancoderkavy/premiere-pro-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
