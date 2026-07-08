## rinku

> Orientation for AI agents and sub-agents working on this Layer 1 network.

# Rinku Agent Guide

Orientation for AI agents and sub-agents working on this Layer 1 network.

## What this repo is

Rinku is a DAG-based distributed ledger with BLS validator consensus, libp2p gossip, WASM contracts, and optional ZK privacy. Critical code lives in:

| Area | Path | Language |
|------|------|----------|
| Protocol primitives | `packages/rinku-core/`, `packages/core/` | Rust + TypeScript (dual impl) |
| Node / consensus | `packages/rinku-node/` | Rust |
| ZK layer | `packages/zk/` | TypeScript + Circom |
| Explorer | `packages/explorer/` | React |
| Validation harness | `scripts/` | TypeScript + Bash |

## CI/CD pipeline

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `rust.yml` | PR/push to main | Unit tests: core, node, contract-sdk; zk feature tests |
| `node.js.yml` | PR/push to main | Build all workspaces; test core + zk |
| `quality.yml` | PR/push to main | `cargo fmt`, `clippy -D warnings`, `cargo audit`, `npm audit`, release build |
| `integration.yml` | PR/push to main | 3-node local consensus + multi-node validation; protocol conformance |
| `coverage.yml` | PR/push to main | Tarpaulin + Vitest → Codecov |
| `network-health.yml` | Cron 15min + manual | Live testnet health + deep protocol validation |

### Running validation locally

```bash
# Unit tests
cargo test --workspace
npm run test:all

# 3-node local integration (CI mode — auto-cleanup)
bash scripts/test-local-3-nodes.sh --ci

# Live testnet validation
npx tsx scripts/validate-testnet.ts https://rinku-genesis.fly.dev
npx tsx scripts/validate-multi-node.ts \
  https://rinku-genesis.fly.dev \
  https://rinku-validator-1.fly.dev \
  https://rinku-validator-2.fly.dev
```

## Agent roles (sub-agent delegation)

| Task type | Delegate to | Skill / rule |
|-----------|-------------|--------------|
| Consensus / merge / slashing changes | `explore` or `generalPurpose` | `.cursor/rules/rust-consensus.mdc` |
| CI / workflow changes | `shell` sub-agent | `.cursor/rules/ci-testing.mdc` |
| Live network validation | `shell` sub-agent | `.cursor/skills/validate-network/SKILL.md` |
| Security review of changes | `security-review` sub-agent | `.cursor/skills/consensus-review/SKILL.md` |
| PR triage / CI fixes | `ci-investigator` sub-agent | — |

## Safety rules for agents

1. **Never commit** `.env`, `*wallet.json`, or validator keys.
2. **Consensus changes** require tests in `packages/rinku-node/tests/` or inline `#[test]`.
3. **Protocol changes** must update both `packages/rinku-core/` (Rust) and `packages/core/` (TS) when they share semantics.
4. **Do not deploy** to Fly.io testnet without explicit user request (`scripts/fly-deploy.sh`).
5. **Stress bots** default to localhost — confirm URLs before running against live testnet.

## Testnet endpoints

- Genesis: `https://rinku-genesis.fly.dev`
- Validator 1: `https://rinku-validator-1.fly.dev`
- Validator 2: `https://rinku-validator-2.fly.dev`

## Known gaps (prioritized backlog)

- [ ] Gated deploy workflow (build → staging → smoke → promote)
- [ ] `cargo-fuzz` targets for encoding / P2P message parsing
- [ ] WASM contract CI (`wasm32-unknown-unknown` builds)
- [ ] Coverage thresholds enforced in CI
- [ ] Chaos/partition simulation in scheduled workflow
- [ ] Third-party security audit (acknowledged in README)

---
> Source: [henry-wrightman/rinku](https://github.com/henry-wrightman/rinku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
