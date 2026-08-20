## phantom-agent-kit

> Give your agent a wallet. Swap, sign, and manage addresses across Phantom's supported chains — and tap into Phantom's docs for context. This plugin also includes skills, rules, and agents to help scaffold wallet-connected apps with the Phantom Connect SDK across React, React Native, and vanilla JS/TS.

# Phantom Connect

Give your agent a wallet. Swap, sign, and manage addresses across Phantom's supported chains — and tap into Phantom's docs for context. This plugin also includes skills, rules, and agents to help scaffold wallet-connected apps with the Phantom Connect SDK across React, React Native, and vanilla JS/TS.

Works with both **Cursor** and **Claude Code**.

## Skill

Use `/phantom-connect` to activate the Phantom Connect skill. It covers all three SDKs (React, React Native, Browser), transaction patterns, payments, token-gating, and NFT minting.

## SDK Rules

### Phantom SDK Best Practices

- Always use `signAndSendTransaction` for sending transactions. NEVER use `signTransaction` — it is NOT supported for embedded wallets.
- In React apps, wrap the application root in `PhantomProvider` with a valid config (`appId`, `providers`, `addressTypes`).
- In vanilla JS/TS apps, initialize a single `BrowserSDK` instance as a singleton. Never create multiple instances.
- Wrap `connect()`, `signAndSendTransaction()`, `signMessage()`, and other async SDK calls in try-catch blocks.
- Check `isConnected` before signing or transaction operations.
- Import `AddressType` from `@phantom/browser-sdk` only.
- Register your app in [Phantom Portal](https://phantom.com/portal) and use a real `appId`.

### Solana Transaction Safety

- Always use `LAMPORTS_PER_SOL` from `@solana/web3.js` for SOL-to-lamport conversion. Never hardcode `1000000000`.
- For time-sensitive transactions, add priority fees with `ComputeBudgetProgram.setComputeUnitPrice()`.
- Verify transaction status after sending with `connection.getSignatureStatus()` or `connection.confirmTransaction()`.
- Use devnet (`https://api.devnet.solana.com`) for testing, mainnet-beta (`https://api.mainnet-beta.solana.com`) for production.
- Never expose private keys in code. Never ask users to input seed phrases or private keys.

### Embedded Wallet Constraints

- `signTransaction` and `signAllTransactions` are NOT supported for embedded wallets. Always use `signAndSendTransaction`.
- Default spending limit: **$1,000 USD per day per app per user**.
- Social login sessions persist **7 days** from the last auth event. Handle session expiration gracefully.
- Social login (Google, Apple) requires a valid `appId` from Phantom Portal.
- React Native apps MUST import `react-native-get-random-values` as the very first import, before anything else.

## MCP Servers

MCP servers are configured in `.mcp.json` at the repo root (shared by both Cursor and Claude Code plugins):

- **phantom-connect-sdk** — Phantom Connect SDK documentation at `https://docs.phantom.com/mcp` for real-time API reference lookups.
- **phantom-mcp** — Wallet operations via [`@phantom/mcp-server`](https://www.npmjs.com/package/@phantom/mcp-server) — sign transactions, transfer tokens, buy/swap tokens, sign messages. Requires a `PHANTOM_APP_ID` environment variable from [Phantom Portal](https://phantom.com/portal).

---
> Source: [phantom/phantom-agent-kit](https://github.com/phantom/phantom-agent-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
