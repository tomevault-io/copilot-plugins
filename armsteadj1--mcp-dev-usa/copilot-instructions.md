## mcp-dev-usa

> Every API your AI agent needs to run a business in the US.

# mcp-dev-usa

Every API your AI agent needs to run a business in the US.
MCP servers for payments, accounting, logistics, communication, banking, and commerce.

## Architecture
- TypeScript monorepo with npm workspaces
- Each MCP server is a standalone package under `packages/<vertical>/<service>/`
- Each server uses `@modelcontextprotocol/sdk` for MCP protocol
- MIT License
- Node 20+

## Verticals & Servers

### Payments
- `packages/payments/stripe` — Charges, customers, subscriptions, Connect marketplace, payment intents, refunds
- `packages/payments/square` — POS, invoices, payments, catalog
- `packages/payments/plaid` — Bank connections, ACH, identity, transactions

### Accounting
- `packages/accounting/quickbooks` — Invoices, expenses, customers, bills, reconciliation
- `packages/accounting/avalara` — Sales tax calculation, exemptions, filing

### Logistics
- `packages/logistics/shippo` — Multi-carrier quotes, labels, tracking, addresses
- `packages/logistics/easypost` — Shipping labels, tracking, rate comparison

### Communication
- `packages/communication/twilio` — SMS, WhatsApp, voice calls
- `packages/communication/sendgrid` — Email campaigns, transactional email
- `packages/communication/resend` — Transactional email (modern alternative)

### Banking
- `packages/banking/mercury` — Business banking, transfers, balances, transactions
- `packages/banking/increase` — ACH, wires, checks, real-time payments

### Commerce
- `packages/commerce/shopify` — Products, orders, inventory, customers, fulfillment
- `packages/commerce/hubspot` — CRM, contacts, deals, companies, pipelines

### Identity (free, no auth)
- `packages/identity/public-data` — USPS address validation, SEC EDGAR company lookups, Census data

## Server Template
Each server follows this structure:
```
packages/<vertical>/<service>/
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts       # MCP server entry point
│   ├── tools/         # Tool definitions
│   └── client.ts      # API client wrapper
└── README.md
```

## Package Naming
- npm scope: `@mcp-dev-usa/<service>`
- Example: `@mcp-dev-usa/stripe`, `@mcp-dev-usa/shippo`

## Key Dependencies
- `@modelcontextprotocol/sdk` — MCP protocol implementation
- `zod` — Schema validation for tool inputs
- Each service's official SDK when available

## The Complete Loop
🛒 Customer places order → 💳 Stripe charges → 📄 QuickBooks invoice + Avalara tax → 📦 Shippo label → 📱 Twilio SMS tracking → 📊 Shopify inventory update → 🏦 Mercury reconciliation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/armsteadj1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
