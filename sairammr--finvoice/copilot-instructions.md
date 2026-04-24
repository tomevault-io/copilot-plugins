## finvoice

> - **Flare TEE** (Confidential Compute): Private invoice data, AI scoring inside TEE

# Finvoice - Private Invoice Factoring on Flare + Hedera

## Architecture
- **Flare TEE** (Confidential Compute): Private invoice data, AI scoring inside TEE
- **Hedera HTS** (SDK only, no Solidity): Attestation NFTs, Receipt NFTs
- **Hedera HCS**: Decentralized event store (replaces traditional DB)
- **Only Solidity**: `tee-extension/contract/InvoiceInstructionSender.sol` on Flare Coston2

## Key Files
- `src/lib/hcs-store.ts` — HCS-backed data store (in-memory cache + HCS replay)
- `src/lib/hedera.ts` — Hedera HTS SDK (NFT operations)
- `src/lib/crypto.ts` — ECIES encrypt/decrypt (Flare secp256k1 + Hedera ED25519)
- `src/lib/contracts.ts` — Flare Coston2 viem client
- `src/lib/scoring.ts` — GPT-4o-mini AI credit scoring
- `src/lib/db.ts` — Re-exports from hcs-store.ts
- `tee-extension/` — Flare TEE extension (handlers + contract)

## Deployed
- Flare contract: `0x616c83ff35d0a9572efb02246ee712a8d062f44b` (Coston2)
- TEE Extension ID: 251 (PRODUCTION)
- Attestation NFT: `0.0.8509042` | Receipt NFT: `0.0.8509043`
- Invoice HCS Topic: `0.0.8511510` | Session HCS Topic: `0.0.8511512`

## Networks
- **Flare**: Coston2, chain ID 114, evm_version `cancun`
- **Hedera**: Testnet, ED25519 operator key, SDK-only

## Auth
- Dynamic.xyz

## Skills
Custom skills in `.claude/skills/`:
- `flare-network`, `flare-tee-extensions`, `flare-smart-accounts`
- `flare-ftso`, `flare-fdc`, `flare-fassets`, `flare-secure-random`
- `hedera-hcs`, `hedera-hts`

## Dev
- `bun run dev` — starts Next.js on port 3000
- `bun run scripts/test-live-e2e.ts` — full E2E test

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sairammr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
