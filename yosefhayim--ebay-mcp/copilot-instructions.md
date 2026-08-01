## ebay-mcp

> Shared repo guidance lives in [AGENTS.md](../AGENTS.md), and full style rules live in [CODE-STYLE.md](../CODE-STYLE.md). Use this file only for Copilot-specific reminders.

# GitHub Copilot Instructions

Shared repo guidance lives in [AGENTS.md](../AGENTS.md), and full style rules live in [CODE-STYLE.md](../CODE-STYLE.md). Use this file only for Copilot-specific reminders.

## Core expectations

- This is a TypeScript/Node.js ESM MCP server for eBay APIs. Keep NodeNext `.js` extensions in imports.
- Use `@/` aliases for imports outside the current folder; use `./sibling.js` only for same-directory imports.
- Hand-written source, test, and UI file basenames are camelCase. Generated specs/types and external docs keep upstream names.
- New or migrated exports use named `export const ... = (...) =>` functions. Do not add default exports.
- Do not hand-edit files under `src/types/`; they are generated from eBay OpenAPI specs.

## Schema and API patterns

- Tool input schemas are Effect-backed. Import runtime helpers from `@/utils/effectSchema.js` and inference/helper types from `@/utils/effectSchemaTypes.js`.
- Do not import `zod` directly outside the schema adapter boundary, and do not add compatibility re-export facades.
- Fallible API work returns `Effect.Effect<Success, TaggedError, Requirements>`. Run Effects at MCP, HTTP, CLI, or test boundaries.
- Do not add endpoint/tool `try/catch` when the work can be represented with Effect.
- Endpoint-backed handlers should validate args, run one endpoint Effect, and return the generated eBay DTO without response reshaping.

## Documentation and tests

- Endpoint-backed API methods need TSDoc with `@param`, `@returns`, a small `@example`, and the official eBay `@see` URL.
- Exported shared utilities document params and returns. Public aliases based on generated response types include `@see`.
- Prefer focused tests near the changed behavior, then run the relevant validation command before claiming the change is done.

## Validation

Use the same gates as `AGENTS.md`:

```bash
npm run typecheck
npm run check:ci
npm test
npm run test:integration
npm run build
```

---
> Source: [YosefHayim/ebay-mcp](https://github.com/YosefHayim/ebay-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
