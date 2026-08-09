## product-discovery-harness

> This repository ships reusable, installable skills for product discovery. Durable

# AGENTS

This repository ships reusable, installable skills for product discovery. Durable
product discovery context belongs in a *target* repository, never in this
repository unless it concerns this harness itself.

Read `ARCHITECTURE.md`, `harness.yml`, and the files in `docs/` before changing
behaviour. Work-item records are under `docs/exec-plans/current/`.

Primary commands:

- `make test` runs the automated suite.
- `make validate` verifies target-contract validation with seeded temporary
  targets and invalid configuration fixtures.
- `python -m product_discovery_harness.cli --help` lists the supported commands.

---
> Source: [nicocirio/product-discovery-harness](https://github.com/nicocirio/product-discovery-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
