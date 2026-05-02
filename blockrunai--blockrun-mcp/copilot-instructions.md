## blockrun-mcp

> MCP server giving Claude real-time data — markets, research, X/Twitter, crypto. Pay per call with USDC.

# BlockRun MCP

MCP server giving Claude real-time data — markets, research, X/Twitter, crypto. Pay per call with USDC.

## Commands

```bash
npm install              # install dependencies
npm run build            # compile with tsup
npm run dev              # watch mode (tsx)
npm start                # start MCP server
npm run typecheck        # type checking
```

## Project structure

```
src/
├── index.ts             # MCP server entry point
├── mcp-handler.ts       # MCP protocol handler
├── tools/               # MCP tool implementations
├── types.ts             # Type definitions
└── utils/               # Shared utilities
```

## Key dependencies

- `@blockrun/llm` — LLM gateway SDK (x402 payments)
- `@modelcontextprotocol/sdk` — MCP protocol SDK
- `zod` — Schema validation
- `viem` — Ethereum interaction
- `qrcode` — Wallet QR code display

## Install in Claude Code

```bash
claude mcp add blockrun npx -y @blockrun/mcp@latest
```

## Conventions

- TypeScript, ESM
- Build with tsup
- Node >= 18
- MIT license
- npm registry: `@blockrun/mcp`

---
> Source: [BlockRunAI/blockrun-mcp](https://github.com/BlockRunAI/blockrun-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
