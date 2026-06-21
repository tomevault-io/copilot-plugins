## savings-goal-tracker

> A monorepo scaffold for the StellarX PH workshop @ PUP QC. Two parts:

# StellarX Workshop Starter — project notes for AI tools

A monorepo scaffold for the StellarX PH workshop @ PUP QC. Two parts:

- `web/` — Next.js 16 + TypeScript + Tailwind v4 frontend (connect Freighter,
  show balances, send a testnet payment, invoke a Soroban contract).
- `contracts/savings-goal/` — a Rust Soroban contract (`init` / `contribute` /
  `get_state`) with unit tests.

## Stack / versions

- `@stellar/stellar-sdk` v15 — use the `rpc` namespace (NOT the old `SorobanRpc`).
- `@stellar/freighter-api` v6 — `signTransaction` returns `{ signedTxXdr, signerAddress }`.
- `soroban-sdk` 22; build target `wasm32v1-none` via `stellar contract build`.
- Network: **testnet** only.

## Testnet reference

| Resource | Value |
|---|---|
| Soroban RPC | `https://soroban-testnet.stellar.org` |
| Horizon | `https://horizon-testnet.stellar.org` |
| Friendbot | `https://friendbot.stellar.org?addr=YOUR_KEY` |
| Network passphrase | `Test SDF Network ; September 2015` |
| USDC issuer (Circle, testnet) | `GBBD47IF6LWK7P7MDEVSCWR7DPUWV3NY3DTQEVFL4NAT4AQH3ZLLFLA5` |
| Explorer | `https://stellar.expert/explorer/testnet` |

## Stellar gotchas (these waste the most time)

1. Use the `rpc` namespace, NOT `SorobanRpc` (v15 SDK).
2. Always **simulate** a Soroban tx before sending (`server.simulateTransaction` → `rpc.assembleTransaction`).
3. `sendTransaction` returning PENDING is NOT success — poll `getTransaction` every 1s for up to 60s.
4. Network passphrase: use `Networks.TESTNET`, never a hardcoded string (wrong one → misleading `tx_bad_auth`).
5. Freighter: **dynamic import only** (`await import('@stellar/freighter-api')`) — static import breaks SSR.
6. Freighter v6: `signTransaction` returns an object → read `.signedTxXdr`.
7. Wrap Freighter calls with a timeout — they can hang if the extension is missing.
8. Trustlines are required before an account can receive a non-native asset (e.g. USDC).
9. Use Soroban RPC for contract calls; use Horizon for balances/history.
10. Soroban i128 args: pass `nativeToScVal(BigInt(x), { type: 'i128' })`.

## Where things live

- Stellar config + Friendbot: `web/src/lib/stellar.ts`
- Balances (Horizon): `web/src/lib/balances.ts`
- Payment build/submit/poll: `web/src/lib/payment.ts`
- Soroban read/write: `web/src/lib/contract.ts`
- Wallet hook: `web/src/hooks/useWallet.ts`
- UI: `web/src/components/*`, wired in `web/src/app/page.tsx`
- Contract: `contracts/savings-goal/src/lib.rs` (+ `test.rs`)
- Deploy: `scripts/deploy.ps1` (Windows) / `scripts/deploy.sh`

---
> Source: [C1D0U/Savings-Goal-Tracker](https://github.com/C1D0U/Savings-Goal-Tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
