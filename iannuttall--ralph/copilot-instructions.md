## ralph

> Keep this file short. It is always loaded into context.

# AGENTS

Keep this file short. It is always loaded into context.

## Build & test
- No build step.
- Tests (dry-run): `npm test`
- Fast real agent check: `npm run test:ping`
- Full real loop: `npm run test:real`

## CLI shape
- CLI entry: `bin/ralph`
- Templates: `.agents/ralph/` (copied to repos on install)
- State/logs: `.ralph/` (local only)
- Skills: `skills/`
- Tests: `tests/`
- Docs/examples: `README.md`, `examples/`

## Quirks / Guardrails
**Add any common quirks guiderails here as needed**

---
> Source: [iannuttall/ralph](https://github.com/iannuttall/ralph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
