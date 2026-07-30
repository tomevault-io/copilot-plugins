## solana-agent-kit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Package Overview

This is `solana-agent-kit` (the core package) - the main SDK for connecting AI agents to Solana protocols. It provides the `SolanaAgentKit` class and framework integrations for LangChain, Vercel AI SDK, and OpenAI Agents.

## Build & Development Commands

```bash
# From this directory (packages/core)
pnpm run build          # Build with tsup (outputs dist/)
pnpm run clean          # Remove dist, .turbo, node_modules
pnpm run test           # Run Jest tests

# From monorepo root
pnpm run build:core     # Build only this package
pnpm run lint           # Lint with Biome
pnpm run lint:fix       # Auto-fix lint issues
```

## Architecture

### Core Exports (`src/index.ts`)

```typescript
export { SolanaAgentKit } from "./agent";
export { createVercelAITools } from "./vercel-ai";
export { createLangchainTools } from "./langchain";
export { createOpenAITools } from "./openai";
export * from "./types";
export * from "./types/wallet";
export * from "./utils/actionExecutor";
export * from "./utils/send_tx";
export * from "./utils/keypairWallet";
```

### SolanaAgentKit Class (`src/agent/index.ts`)

The main class uses TypeScript generics for type-safe plugin composition:

```typescript
class SolanaAgentKit<TPlugins = Record<string, never>> {
  connection: Connection;      // Solana RPC connection
  config: Config;              // API keys and settings
  wallet: BaseWallet;          // Solana wallet interface
  evmWallet?: EvmWallet;       // Optional EVM wallet for bridging
  methods: TPlugins;           // Type-safe plugin methods
  actions: Action[];           // AI agent tools

  use<P extends Plugin>(plugin: P): SolanaAgentKit<TPlugins & PluginMethods<P>>
}
```

The `.use()` method merges plugin methods into `agent.methods` with full type inference.

### Plugin Interface (`src/types/index.ts`)

```typescript
interface Plugin {
  name: string;
  methods: Record<string, any>;  // Programmatic API functions
  actions: Action[];             // AI tools with Zod schemas
  initialize(agent: SolanaAgentKit): void;
}

interface Action {
  name: string;
  similes: string[];             // Alternative trigger phrases
  description: string;
  examples: ActionExample[][];
  schema: z.ZodType<any>;        // Input validation
  handler: Handler;              // (agent, input) => Promise<result>
}
```

### Wallet Interfaces (`src/types/wallet.ts`)

- `BaseWallet`: Solana wallet (signTransaction, signAllTransactions, signAndSendTransaction, signMessage)
- `EvmWallet`: EVM wallet for cross-chain operations
- `KeypairWallet`: Default implementation using Solana Keypair (`src/utils/keypairWallet.ts`)

### AI Framework Adapters

- `src/vercel-ai/index.ts` - `createSolanaTools()` for Vercel AI SDK
- `src/langchain/index.ts` - `createLangchainTools()` for LangChain
- `src/openai/index.ts` - `createOpenAITools()` for OpenAI Agents SDK

All adapters convert `Action[]` to framework-specific tool formats with Zod schema validation.

### Utilities

- `src/utils/actionExecutor.ts` - Validates input with Zod then calls handler
- `src/utils/send_tx.ts` - Transaction sending with priority fees
- `src/utils/getMintInfo.ts` - SPL token mint information helpers

## File Structure

```
src/
├── index.ts              # Package exports
├── agent/index.ts        # SolanaAgentKit class
├── types/
│   ├── index.ts          # Plugin, Action, Config, response types
│   ├── action.ts         # Action helper types
│   └── wallet.ts         # BaseWallet, EvmWallet interfaces
├── vercel-ai/index.ts    # Vercel AI SDK adapter
├── langchain/index.ts    # LangChain adapter
├── openai/               # OpenAI Agents SDK adapter
│   ├── index.ts
│   └── utils.ts
├── utils/
│   ├── actionExecutor.ts # Action execution with validation
│   ├── keypairWallet.ts  # KeypairWallet implementation
│   ├── send_tx.ts        # Transaction helpers
│   ├── getMintInfo.ts    # Token mint utilities
│   └── zod.ts            # Zod schema helpers
└── constants/index.ts    # Shared constants
```

## Code Style

- **Formatting**: Biome (not ESLint/Prettier)
- **Modules**: ESM (`"type": "module"`)
- **Output**: Dual format (CJS + ESM) via tsup
- **Naming**: camelCase functions, PascalCase types

## Key Patterns

1. **Plugin methods bound to plugin context**: Methods are called with `method.bind(plugin)` so `this` refers to the plugin

2. **Action handlers receive agent**: `handler(agent: SolanaAgentKit, input: Record<string, any>)`

3. **Config holds all API keys**: Check `Config` interface in `src/types/index.ts` for available keys

4. **Actions have 128 tool limit**: Framework adapters warn if exceeding this limit

---
> Source: [sendaifun/solana-agent-kit](https://github.com/sendaifun/solana-agent-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
