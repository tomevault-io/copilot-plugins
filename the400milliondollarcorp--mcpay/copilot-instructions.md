## mcpay

> - **Install**: `pnpm install` (requires pnpm@9.12.0+, node@20.10.0+)

# MCPay Agent Guide

## Build/Lint/Test Commands
- **Install**: `pnpm install` (requires pnpm@9.12.0+, node@20.10.0+)
- **Build**: `pnpm build` (builds all apps/packages via turbo)
- **Typecheck**: `pnpm typecheck` (checks all packages)
- **Lint**: `pnpm lint` (lints all packages)
- **Test**: `pnpm test` (runs all tests)
- **Dev**: `pnpm dev` (runs apps/* and packages/* in parallel)
- **Single package**: `pnpm -C packages/js-sdk build` or `turbo run build --filter=mcpay`

## Architecture
MCPay is a monorepo enabling on-chain payments for MCP servers via x402 protocol. Key apps: `app/` (Next.js registry/dashboard), `facilitator/` (Hono payment facilitator), `mcp/` `mcp2/` `mcp-data/` (MCP servers), `api2/` (API layer). Main package: `packages/js-sdk/` (mcpay SDK + CLI). Uses Turbo for builds, Next.js 15, Drizzle ORM for database, Hono for HTTP handlers, viem for EVM, and x402 for payment protocol.

## Code Style & Conventions
- **TypeScript strict mode**: All code uses strict typing (target ES2022, module ESNext, moduleResolution Bundler)
- **Imports**: Use ESM syntax (`import/export`), file extensions `.js` in imports (e.g., `'./utils/signer.js'`)
- **Naming**: camelCase for functions/variables, PascalCase for types/components
- **Exports**: Named exports preferred (avoid default exports except Next.js pages/handlers)
- **No tests**: No test framework currently configured; add tests only when tooling exists
- **Error handling**: Use explicit error returns or throw with typed errors
- **Formatting**: No prettier/eslint config; follow existing code patterns
- **React**: Next.js App Router, TypeScript, Tailwind CSS, Radix UI components

---
> Source: [The400MillionDollarCorp/mcpay](https://github.com/The400MillionDollarCorp/mcpay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
