## qbtc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**qbtc** is a Cosmos SDK blockchain that lets Bitcoin holders claim tokens by proving ownership of their Bitcoin addresses using zero-knowledge proofs (PLONK). It supports P2PKH and P2WPKH Bitcoin addresses. The ZK approach is TSS/MPC-compatible — users only need a signature (r, s, pubkey), not direct private key access.

## Commands

```bash
# Build all binaries (qbtcd, bifrost, utxo-indexer)
make build

# Build zkprover CLI only
make build-prover

# Run unit tests (excludes ZK circuit tests)
make test-unit

# Run all tests including ZK (requires trusted setup)
make test-all

# Run ZK-specific tests
make test-zk

# Run a single test
go test ./x/qbtc/zk/... -run TestFoo -v -tags testing

# Lint
make lint
make lint-fix

# Full check (proto format + lint + markdown lint)
make check

# Generate protobuf
make proto-gen

# Generate trusted setup (Hermez PoT ceremony)
make setup-prover

# Start a local node
./scripts/start-node.sh
```

**Note**: ZK circuit tests require the `-tags testing` build tag and a pre-generated trusted setup. Use `make test-unit` for fast iteration without ZK tests.

## Architecture

### System Flow

```
Off-chain:
  TSS/MPC Signer → ECDSA signature
  zkprover CLI   → PLONK proof (hides sig & pubkey)
                          ↓
On-chain (x/qbtc module):
  MsgClaimWithProof → ZK Verifier → BankKeeper.MintCoins
                                  → mark UTXO as claimed
Background:
  Bifrost (P2P) → observes Bitcoin blocks → indexes UTXOs via gRPC
```

### Key Packages

| Package | Role |
|---|---|
| `app/` | Cosmos SDK app wiring; all module keepers connected here |
| `x/qbtc/keeper/` | Core state: UTXOs, claims, ZK verifying key; handles `MsgClaimWithProof` |
| `x/qbtc/zk/` | ZK proof system: five circuits, PLONK verifier, trusted setup |
| `x/qbtc/ebifrost/` | EnshrinedBifrost — P2P gossip embedded inside the validator node |
| `bifrost/` | Standalone bifrost daemon (separate process, same P2P protocol) |
| `bitcoin/` | Bitcoin RPC client for UTXO indexing |
| `cmd/zkprover/` | CLI tool users run to generate proofs locally |
| `cmd/proof-service/` | HTTP service wrapper around zkprover for remote proof generation |

### ZK Circuit Design

`BTCPubKeyOwnershipCircuit` in `x/qbtc/zk/` proves P2PKH / P2WPKH ownership via ECDSA secp256k1 + Hash160. It uses **PLONK + KZG commitments on BN254**. The trusted setup uses Hermez/Polygon Powers of Tau (production-grade). Proofs are bound to `(destination address, chain ID, version)` to prevent replay and front-running.

The ZK verifying key is loaded at genesis init and stored immutably in chain state — it cannot be replaced post-genesis.

### Bifrost / P2P

The `EnshrinedBifrost` (`x/qbtc/ebifrost/`) runs as an embedded service inside each validator, using libp2p for gossip. A separate standalone `bifrost` daemon (`bifrost/`) can also be run for distributed Bitcoin block observation. Both use the same P2P protocol and submit observed Bitcoin block data via gRPC to `qbtcd`.

### Claim Message Flow

`MsgClaimWithProof` →
1. `ValidateBasic()` — syntax checks
2. `zk.Verifier.VerifyProof()` — PLONK verification against stored VK
3. `Keeper.Utxoes.Get()` — look up UTXO(s) and check `EntitledAmount > 0`
4. `BankKeeper.MintCoins()` — mint to claimer's address
5. Set `EntitledAmount = 0` on the UTXO (idempotency guard)

### Dependencies

Uses btcq-org forks of `cosmos-sdk`, `cometbft`, and `wasmd` for IBC v10 / CosmWasm compatibility. See `go.mod` for exact versions. Go 1.25+ required.

## Further Reading

- `docs/ZK_SYSTEM.md` — detailed ZK circuit spec, constraint counts, and security model
- `genesis.md` — genesis configuration

---
> Source: [btcq-org/qbtc](https://github.com/btcq-org/qbtc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
