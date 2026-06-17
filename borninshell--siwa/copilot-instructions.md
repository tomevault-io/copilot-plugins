## siwa

> Wallet-based authentication for autonomous AI agents. Like "Sign In With Google" but for agents — no browsers, no OAuth, just cryptography.

# SIWA - Sign In With Agent

Wallet-based authentication for autonomous AI agents. Like "Sign In With Google" but for agents — no browsers, no OAuth, just cryptography.

## When to Use

Use SIWA when you need to:
- Authenticate with services that support SIWA
- Prove your agent identity using your Solana keypair
- Make authenticated API calls without API keys or OAuth
- Integrate payments with authentication (via x402)

## Quick Start

```typescript
import { SIWAClient } from '@siwa/client';
import { Keypair } from '@solana/web3.js';

// Use your agent's existing keypair
const client = new SIWAClient({
  keypair: Keypair.fromSecretKey(yourSecretKey),
});

// Authenticate with any SIWA-enabled service
const session = await client.signIn('https://api.example.com');

// Make authenticated requests
const response = await client.authenticatedFetch(session)('/api/data');
```

## Installation

```bash
npm install @siwa/client @solana/web3.js
```

## Packages

| Package | Purpose |
|---------|---------|
| `@siwa/core` | Message parsing and verification |
| `@siwa/client` | Agent SDK with signIn() |
| `@siwa/server` | Server utilities and middleware |
| `@siwa/x402` | Micropayment integration |

## How It Works

1. **Request Challenge** — Agent sends public key to service
2. **Sign Message** — Agent signs challenge with private key
3. **Verify & Session** — Service verifies, issues token
4. **Authenticated Calls** — Agent uses token for API calls

## Full Documentation

See [agents.md](./agents.md) for complete API reference and examples.

## Resources

- **Demo**: https://siwa-landing-black.vercel.app/demo.html
- **GitHub**: https://github.com/borninshell/siwa
- **Website**: https://siwa-landing-black.vercel.app

## License

Apache 2.0

---

Built by [The Shellborn Collective](https://shellborn.io) 🦞

---
> Source: [borninshell/siwa](https://github.com/borninshell/siwa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
