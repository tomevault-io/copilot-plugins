## 05-product-integrator

> Use when wiring protocol to app, SDK integration, escrow flow, end-to-end connection, ZeroDev smart accounts, cofhejs encryption


# Integrator — Protocol to App

> **Read before acting:**
> - `.claude/docs/product/PROTOCOL_INTEGRATION.md`
> - `.claude/docs/product/ARCHITECTURE.md`

Wire the three repos together: reineira-code (contracts) → backend → frontend.

## Integration Stack

```
User → Frontend (React 19) → Backend (TypeScript) → Protocol (Arbitrum)
                ↕                    ↕                  ↕
        ZeroDev SDK          @reineira/sdk      ConfidentialEscrow
        cofhejs (encrypt)    Business logic     Fhenix CoFHE
```

## SDK Usage

```typescript
import { ReineiraSDK } from '@reineira-os/sdk';

const sdk = ReineiraSDK.create({
  network: 'testnet',
  privateKey: process.env.PRIVATE_KEY,
});
await sdk.initialize();

const escrow = await sdk.escrow
  .build()
  .amount(sdk.stablecoin(1000))
  .owner('0xRecipient...')
  .condition('0xYourResolver...', resolverData)
  .create();
```

## Integration Checklist

### Frontend → Backend
- [ ] Auth flow (JWT via authStore)
- [ ] API service classes wrapping endpoints
- [ ] Error handling with user-friendly messages

### Backend → Protocol
- [ ] SDK initialization with signer
- [ ] Escrow creation endpoint
- [ ] Escrow status polling / webhook
- [ ] Proof submission endpoint (if zkTLS)

### Frontend → Protocol (direct)
- [ ] ZeroDev smart account for user operations
- [ ] cofhejs for FHE encryption (client-side)
- [ ] Passkey auth flow

### Testing

```
1. Start backend: pnpm dev:backend
2. Start frontend: pnpm dev:app
3. Testnet: Arbitrum Sepolia
4. Create escrow → trigger condition → verify release
```

## Checklist

- [ ] Full flow works end-to-end on testnet
- [ ] Error states handled at each boundary
- [ ] Environment variables documented
- [ ] No hardcoded addresses

---
> Source: [ReineiraOS/reineira-atlas](https://github.com/ReineiraOS/reineira-atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
