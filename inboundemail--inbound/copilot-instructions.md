## primary

> - Only use bun for package manager

Project Rules:

- Only use bun for package manager
- Never run `bun run dev` or `bun run build` without my approval
- Before every message say "Rules Loaded: Primary (any others)"
- Never run npx tsc or any other build/type checker without my approval first — this will break the project
- `receivedEmails` and `parsedEmails` are deprecated — always use `structuredEmails` instead
- Never create duplicate type definitions; always find/extend an existing type from the schema
- All icons use Nucleo via MCP
- New functions and components go in the API folder — follow the correct API spec
- Do not run AWS CLI list commands; give me the command and I'll run it

---
> Source: [inboundemail/inbound](https://github.com/inboundemail/inbound) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
