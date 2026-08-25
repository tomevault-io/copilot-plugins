## stock-simulator

> New here? Read [docs/OVERVIEW.md](docs/OVERVIEW.md) first — what this project is, who

# CuckooTrade (stock_simulator)

New here? Read [docs/OVERVIEW.md](docs/OVERVIEW.md) first — what this project is, who
it's for, current state, and the settled decision log. The active build contract is
[docs/V1_SPEC.md](docs/V1_SPEC.md). Infrastructure/deploy runbook:
[docs/QUICK_START.md](docs/QUICK_START.md). The root README is the public product pitch.

Critical: **pushing to `main` deploys to production** — CI builds the touched
service's image and ArgoCD syncs it to the live EKS cluster within minutes. Work on
feature branches; merge to `main` only to ship.

---
> Source: [tj-miller-dev/stock_simulator](https://github.com/tj-miller-dev/stock_simulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
