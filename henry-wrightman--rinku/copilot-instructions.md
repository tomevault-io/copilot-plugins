## ci-testing

> CI/CD and test harness conventions for Rinku


# CI & Testing Harness

## Workflow layout

| File | Add tests here when... |
|------|------------------------|
| `rust.yml` | New Rust crate or feature-gated module |
| `node.js.yml` | New TS workspace with tests |
| `quality.yml` | New security/lint requirements |
| `integration.yml` | New multi-node or conformance checks |
| `network-health.yml` | New live testnet probes |

## Integration test entry points

```bash
# CI-mode 3-node test (auto-cleanup)
bash scripts/test-local-3-nodes.sh --ci

# Manual local testnet
bash scripts/local-testnet.sh start
bash scripts/local-testnet.sh validate
```

## Adding a new validation script

1. Place in `scripts/`, use `npx tsx` (not ts-node)
2. Exit `0` on success, `1` on failure
3. Wire into `integration.yml` or `network-health.yml`
4. Add npm script alias in root `package.json` if user-facing

## CI constraints

- Integration job timeout: 25 minutes
- Use `npm ci` not `npm install` in workflows
- Release builds use `cargo build --release -p rinku-node` (matches `Dockerfile.fly`)
- Never store secrets in workflow files — use GitHub Secrets

---
> Source: [henry-wrightman/rinku](https://github.com/henry-wrightman/rinku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
