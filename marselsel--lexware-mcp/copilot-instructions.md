## lexware-mcp

> This is an open-source MCP server for the Lexware Office API, built with the Skybridge

This is an open-source MCP server for the Lexware Office API, built with the Skybridge
framework. When planning or updating the codebase, use the `skybridge` skill.

Key conventions:
- Keep all Lexware logic (`src/lexware`, `src/config.ts`, `src/tools`) independent of Skybridge.
- Tools are gated by capability tiers in `src/config.ts` and registered conditionally in
  `src/tools/index.ts` (read = always; drafts/finalize = env-gated).
- Finalizing is irreversible: any legally-binding write must be a separate, confirmation-gated
  finalize-tier tool — never a flag on a draft tool.
- `npm run build` typechecks; `npm test` runs vitest. Never log secrets or PII.

---
> Source: [marselsel/lexware-mcp](https://github.com/marselsel/lexware-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
