## invoice-ninja-mcp

> TypeScript MCP server for Invoice Ninja v5 API. 26 tools across 7 categories (system, invoices, clients, products, payments, quotes, expenses). Primary use case: creating and managing invoices through AI assistants.

# Invoice Ninja MCP Server

TypeScript MCP server for Invoice Ninja v5 API. 26 tools across 7 categories (system, invoices, clients, products, payments, quotes, expenses). Primary use case: creating and managing invoices through AI assistants.

## Specifications

- [API Integration](specs/api-integration.md) - HTTP client, auth, error handling
- [Invoices](specs/invoices.md) - Invoice CRUD, email, bulk actions (primary feature)
- [Clients](specs/clients.md) - Client and contact management
- [Products](specs/products.md) - Product catalog (read-only)
- [Payments](specs/payments.md) - Payment recording
- [Quotes](specs/quotes.md) - Quote management and conversion to invoices
- [Expenses](specs/expenses.md) - Expense tracking
- [Distribution](specs/distribution.md) - npm/npx packaging and install instructions

## Research

- [Existing MCP Servers](research/existing-mcp-servers.md) - Competitive analysis

## Architecture

- Tool factory pattern with `createTool` helper (lazy evaluation)
- `ToolDefinition` uses `any` default for Args generic to avoid union type issues
- `registerGroup()` processes each tool category separately
- Native fetch, no HTTP library dependencies
- Zod schemas for all tool input validation

## Development

```bash
pnpm install
pnpm run build
pnpm run watch  # dev mode
```

## Environment Variables

- `INVOICE_NINJA_URL` - Base URL of Invoice Ninja instance
- `INVOICE_NINJA_API_TOKEN` - API token from Settings > Account Management > API Tokens

---
> Source: [Fuciuss/invoice-ninja-mcp](https://github.com/Fuciuss/invoice-ninja-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
