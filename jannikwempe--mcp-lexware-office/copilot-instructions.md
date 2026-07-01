## mcp-lexware-office

> Lexware Office REST API: https://developers.lexoffice.io/docs/

# Lexware Office MCP Server — Claude Instructions

## API Documentation

Lexware Office REST API: https://developers.lexoffice.io/docs/

Fetch specific sections on demand with WebFetch. Key sections:
- Contacts: `/docs/#contacts-endpoint`
- Invoices: `/docs/#invoices-endpoint`
- Vouchers (voucherlist): `/docs/#voucherlist-endpoint`
- Down-Payment Invoices: `/docs/#down-payment-invoices-endpoint`
- Dunnings: `/docs/#dunnings-endpoint`

## Project Conventions

- All tools live in `src/index.ts` (single file, ~1800+ lines)
- List tools use `/v1/voucherlist?voucherType=<type>` with `status`, `contactId`, `page`, `size` params
- Detail tools use `/v1/<resource>/{id}` directly
- Write tools use get-before-put for contacts (PUT overwrites entire resource)
- Optional fields use `!== undefined` guards, not falsy checks
- No test framework — `pnpm run build` (tsc) is the verification step

## What NOT to commit

- `docs/` — internal Claude planning files (.gitignore already excludes this)

---
> Source: [JannikWempe/mcp-lexware-office](https://github.com/JannikWempe/mcp-lexware-office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
