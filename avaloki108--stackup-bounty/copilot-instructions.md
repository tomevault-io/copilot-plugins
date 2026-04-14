## stackup-bounty

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a bug bounty workspace for Stackup's smart contract protocols. Contains two independent Foundry projects:

| Project | Path | Description |
|---------|------|-------------|
| **Keystore** | `keystore/` | Merkle tree-based configuration management for ERC-4337 smart accounts |
| **Forwarding Address** | `forwarding-address/` | Deterministic address generation for payment attribution |

## Build & Test Commands

Both projects use Foundry with identical commands:

```bash
# From project directory (keystore/ or forwarding-address/)
forge install          # Install dependencies
forge build            # Compile contracts
forge test             # Run all tests
forge test --mt <name> # Run specific test by name
forge test -vvv        # Verbose with traces
```

Keystore also has npm dependencies:
```bash
cd keystore && npm install
npm run examples:verify-ucmt  # Generate/verify Merkle tree example
```

## Architecture

### Keystore Protocol

The Keystore enables gas-efficient, privacy-preserving account configuration through a User Configuration Merkle Tree (UCMT):

```
src/
├── core/Keystore.sol           # Singleton storing root hashes, handles updates & validation
├── account/
│   ├── KeystoreAccount.sol     # ERC-4337 account using Keystore for validation
│   └── KeystoreAccountFactory.sol
├── verifier/                   # Stateless signature verification
│   ├── UserOpECDSAVerifier.sol
│   ├── UserOpMultiSigVerifier.sol
│   ├── UserOpWebAuthnVerifier.sol
│   └── UserOpWebAuthnCosignVerifier.sol
├── interface/
│   ├── IKeystore.sol
│   └── IVerifier.sol
└── lib/
    ├── Actions.sol             # UpdateAction, ValidateAction structs
    ├── KeystoreUserOperation.sol
    └── OnlyKeystore.sol        # Modifier for verifier access control
```

Key concepts:
- **UCMT nodes**: `abi.encodePacked(verifier, config)` - 20 bytes verifier address + arbitrary config
- **refHash**: Permanent reference to a user's configuration, used for deterministic account addresses
- **rootHash**: Current Merkle root, can be updated via `handleUpdates()`
- **2D Nonce**: `uint192 key | uint64 sequence` for cross-chain replay protection

Validation flow: Account -> Keystore.validate() -> Verifier.validateData() -> returns SIG_VALIDATION_SUCCESS (0) or SIG_VALIDATION_FAILED (1)

### Forwarding Address Protocol

Minimal proxy pattern for counterfactual payment addresses:

```
src/
├── ForwardingAddress.sol       # Receives funds, sweeps to receiver
└── ForwardingAddressFactory.sol # CREATE2 factory, sweepFor() convenience
```

Address derivation: `keccak256(abi.encode(receiver, salt))` -> deterministic clone address

## Code Style

- Solidity 0.8.28, optimizer 1,000,000 runs
- Internal functions: `_prefixed`
- Immutable storage: `_prefixed`
- Constructor params: `aParam` prefix when matching storage names
- Fuzz tests: `testFuzz_` prefix
- Use Solady for gas efficiency (LibBytes, MerkleProofLib, ECDSA, WebAuthn, LibClone)
- ReentrancyGuardTransient for state-modifying external calls

## Key Files for Security Review

**Keystore critical paths:**
- `Keystore.handleUpdates()` - Root hash updates with Merkle proof verification
- `Keystore.validate()` - Transaction validation entry point
- `Keystore._fetchOrValidateNode()` - Cache vs proof validation logic
- `KeystoreAccount._validateSignature()` - ERC-4337 signature handling

**Forwarding Address critical paths:**
- `ForwardingAddressFactory.sweepFor()` - Deploy + sweep in single call
- `ForwardingAddress.sweep()` - ETH/ERC20 transfer logic

## Specification

Full Keystore protocol spec: `keystore/doc/spec.md`

## Deployed Contracts

**Keystore:**
- Keystore: `0x69C9F626b5Bd934C0F9806346682eD407FB978d3`
- KeystoreAccountFactory: `0x625cF8EDec3f68d48D3aA385F356524B04760BE8`

**Forwarding Address:**
- ForwardingAddressFactory: `0xb15dcf07aE4bA9423c9ee0Aba58773C9fd7ec293`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/avaloki108)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/avaloki108)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
