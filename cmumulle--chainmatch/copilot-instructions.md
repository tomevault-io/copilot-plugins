## chainmatch

> Online tennis game with mouse-driven aiming, hero selection, and

# CLAUDE.md

## Project: ACE CHAIN (chainmatch)

Online tennis game with mouse-driven aiming, hero selection, and
blockchain-verified match results on Base (EVM).

## Specifications

Read these files before starting any task:
- `SPEC-PROTOTYPE-v0.1.md` — Prototype scope, game design, acceptance criteria
- `SPEC-CHAIN-v0.1-BASE.md` — Solidity contract, EIP-712, Base network details
- `TASKS.md` — Task breakdown (work through in order: T-001, T-002, ...)
- `TESTPLAN.md` — Test plan and verification commands

## Current Phase

Working on: Phase 3 (AI & Match Loop), starting at T-023.

Completed: Phase 0 (T-001 through T-006), Phase 1 (T-007 through T-012), Phase 2 (T-013 through T-022).

## Rules

- Follow TASKS.md sequentially. Do not skip ahead.
- After completing each task, run its test command and confirm it passes.
- Mark completed tasks in TASKS.md by changing `[ ]` to `[x]`.
- Commit after each completed task with message: `T-XXX: <task title>`
- Use Rust + Bevy for game client, Foundry for Solidity contracts.
- All shared types go in `ace-shared` crate.
- Do not add features not listed in SPEC-PROTOTYPE-v0.1.md.
- If a task is ambiguous, re-read the relevant spec section before asking.

## Tech Stack

- Rust (latest stable)
- Bevy 0.15+
- Foundry (forge/cast) for Solidity
- alloy for Rust ↔ EVM interaction
- Base Sepolia (chain ID 84532) for testnet

## Build Commands
```
cargo build --workspace          # Build all Rust crates
cargo test --workspace           # Run all Rust tests
cargo run -p ace-client          # Run game client
cd contracts && forge build      # Build Solidity
cd contracts && forge test -vv   # Run Solidity tests
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cmumulle)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cmumulle)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
