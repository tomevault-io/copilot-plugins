## stablecoin-scaffold

> Guidance for AI coding agents (Claude Code, Codex, Cursor, Copilot, and others) working in this

# AGENTS.md

Guidance for AI coding agents (Claude Code, Codex, Cursor, Copilot, and others) working in this
repository. Loaded into agent context automatically — keep it concise.

## Overview

Cadence tutorial scaffold showing how to build a `PiggyBank` contract that accepts deposits of the
USDF stablecoin on Flow. Ships with a `USDF_MOCK` contract used on the emulator and in the Cadence
test framework, plus transactions and scripts that target the same contract name on mainnet
(`1e4aa0b87d10b141`). No JavaScript, Go, or build tooling — the repo is driven entirely through
the Flow CLI (`flow emulator`, `flow project deploy`, `flow transactions send`, `flow scripts
execute`, `flow test`). See `README.md` for the end-to-end tutorial walkthrough.

## Build and Test Commands

Flow CLI must be installed (https://developers.flow.com/build/tools/flow-cli). There is no
Makefile, `package.json`, or other build manifest in this repo.

- `flow emulator` — start the local emulator at `127.0.0.1:3569` (per `flow.json` networks).
- `flow dependencies install` — materialize mainnet-sourced dependencies from `flow.json` into
  `imports/` (required on a fresh clone before first deploy).
- `flow project deploy` — deploy `EVMVMBridgedToken_2aabea2058b5ac2d339b163c6ab6f2b6d53aabed`
  (source `cadence/contracts/USDF_MOCK.cdc`) and `PiggyBank` to the emulator account
  `f8d6e0586b0a20c7` (per `flow.json` `deployments.emulator`).
- `flow test cadence/tests/PiggyBank_test.cdc` — run the Cadence test suite.
- `flow transactions send cadence/transactions/SetupUSDFMockVault.cdc --signer emulator-account`
  — provision a USDF vault on the signer.
- `flow transactions send cadence/transactions/MintUSDFMock.cdc <amount> <recipient> --signer emulator-account`
  — mint up to 1000.0 USDF per call (mock-only; see pre-condition in `USDF_MOCK.cdc:140`).
- `flow transactions send cadence/transactions/DepositToPiggyBank.cdc <amount> --signer emulator-account`
- `flow transactions send cadence/transactions/WithdrawFromPiggyBank.cdc <amount> --signer emulator-account`
- `flow scripts execute cadence/scripts/GetPiggyBankBalance.cdc`
- `flow scripts execute cadence/scripts/GetUserUSDFBalance.cdc <address>`
- `flow scripts execute cadence/scripts/GetUSDFMockBalance.cdc <address>`
- `flow scripts execute cadence/scripts/GetUSDFMockInfo.cdc`

## Architecture

```
cadence/
  contracts/
    PiggyBank.cdc       — shared vault; deposit() / withdraw() / getBalance(); emits Deposited, Withdrawn
    USDF_MOCK.cdc       — contract EVMVMBridgedToken_2aabea2058b5ac2d339b163c6ab6f2b6d53aabed
                          implementing FungibleToken; public mintTokens() capped at 1000.0 per call
  transactions/
    SetupUSDFMockVault.cdc   — creates /storage/...Vault + /public/...Vault + /public/...Receiver
    MintUSDFMock.cdc         — mints via public mintTokens(); deposits to recipient receiver
    DepositToPiggyBank.cdc   — withdraws from signer's USDF vault into PiggyBank
    WithdrawFromPiggyBank.cdc — withdraws from PiggyBank into signer's USDF vault
  scripts/
    GetPiggyBankBalance.cdc  — returns PiggyBank.getBalance()
    GetUserUSDFBalance.cdc   — borrows &Vault at /public/...Vault (panics if missing)
    GetUSDFMockBalance.cdc   — same as above but returns 0.0 if vault is missing
    GetUSDFMockInfo.cdc      — reads FTDisplay metadata view + totalSupply
  tests/
    PiggyBank_test.cdc  — Cadence Test framework; deploys both contracts, exercises deposit/withdraw
flow.json               — contracts, dependencies, network aliases, emulator deployment
DOCS.md                 — tutorial on building network-compatible mock contracts
README.md               — end-to-end emulator walkthrough
```

Deployment order (see `cadence/tests/PiggyBank_test.cdc:8-23`): USDF mock first, then `PiggyBank`
(its `init()` calls `createEmptyVault` on the USDF contract).

## Conventions and Gotchas

- **Contract is registered in `flow.json` under its mainnet name**, not its filename. The entry
  `EVMVMBridgedToken_2aabea2058b5ac2d339b163c6ab6f2b6d53aabed` sources
  `cadence/contracts/USDF_MOCK.cdc`. Every `import` uses the long name, and storage/public paths
  embed it literally (e.g. `/storage/EVMVMBridgedToken_2aabea2058b5ac2d339b163c6ab6f2b6d53aabedVault`).
  Do not rename the contract or the paths — interface compatibility with the real mainnet USDF
  depends on byte-for-byte identical names and paths (see `DOCS.md`).
- **flow.json alias reality** (verify before claiming cross-network support):
  - `EVMVMBridgedToken_2aabea2058b5ac2d339b163c6ab6f2b6d53aabed`: `emulator` =
    `f8d6e0586b0a20c7`, `mainnet` = `1e4aa0b87d10b141`, `testing` = `0000000000000007`.
    **No `testnet` alias is configured** despite what `README.md:11` says; scripts/transactions
    pointed at `--network testnet` will fail to resolve this import.
  - `PiggyBank`: only a `testing` alias (`0000000000000007`). Not deployed to mainnet or testnet.
- **Public mint is mock-only.** `USDF_MOCK.cdc` exposes `access(all) fun mintTokens(...)` with a
  1000.0 per-call cap. The real mainnet USDF does not expose public mint; `MintUSDFMock.cdc` will
  not work against `--network mainnet`.
- **Hard-coded paths in scripts/transactions, not contract constants.** `DOCS.md` (and every
  script/transaction in the repo) uses literal path expressions like
  `/public/EVMVMBridgedToken_2aabea2058b5ac2d339b163c6ab6f2b6d53aabedVault` rather than
  `USDF.VaultPublicPath` so the same file works against the mainnet contract, which may not
  expose those constants.
- **Token decimals = 6**, symbol = `USDF`, name = `USDF MOCK` (see `USDF_MOCK.cdc:229-231`).
  `GetUSDFMockInfo.cdc:27` hard-codes `decimals: 6` because the script cannot rely on a
  `getDecimals()` method existing on mainnet.
- **Cadence tests live in `cadence/tests/` and use relative paths** (`../contracts/USDF_MOCK.cdc`)
  inside `Test.deployContract`. Run them from the repo root with `flow test`.
- **Dependencies are mainnet-sourced** in `flow.json` (`FungibleToken`, `MetadataViews`, `Burner`,
  `EVM`, etc.). `flow dependencies install` (or `flow project deploy` on a fresh clone) will
  materialize them into an `imports/` directory; `.gitignore` keeps the top-level `imports/`
  folder untracked but `.cursorignore` re-includes it for editor context.

## Files Not to Modify

- `emulator-account.pkey` — emulator signing key (gitignored; generated by `flow emulator`).
- `imports/` — Flow CLI-materialized dependency cache (gitignored).
- Contract name `EVMVMBridgedToken_2aabea2058b5ac2d339b163c6ab6f2b6d53aabed` and its storage/public
  paths in `USDF_MOCK.cdc` — changing them breaks mainnet interface parity.

---
> Source: [onflow/stablecoin-scaffold](https://github.com/onflow/stablecoin-scaffold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
