## cl-mycelium

> This repository implements cl-mycelium, the organismic coordination layer for trusted Core Lightning fleets.

# AGENTS.md — cl-mycelium

## Project identity

This repository implements cl-mycelium, the organismic coordination layer for trusted Core Lightning fleets.

cl-mycelium is implemented as part of the existing cl-hive plugin surface. It coordinates, senses, remembers, computes organism state, and emits bounded hints/repair prompts.

## Core invariants

- cl-mycelium is advisory.
- cl-mycelium must not directly spend funds.
- cl-mycelium must not directly execute rebalances.
- cl-mycelium must not directly open or close channels.
- cl-mycelium must not directly set channel fees.
- `cl_revenue_ops` remains the local executor.
- `["hive","hints"]` remains the compatibility surface.
- M2 influence must remain scoped, explicit, reversible, and test-gated.
- Production M2 default scope must not be `all_hints`.
- No Sling dependency.
- No hidden behavior changes.

## Current organism model

cl-mycelium should implement a bounded Lightning organism:

- body: fleet nodes, channels, capital
- environment: Lightning graph
- senses: interoception, proprioception, exteroception
- stress: deviation from viable target morphology
- metabolism: fees, costs, reserves, net usable energy
- target morphology: remembered desired fleet/channel form
- immune/pathology: toxic paths, bad peers, closure-watch
- bioprompting: hints to local competent executors
- executor: `cl_revenue_ops`

## Required reading before tasks

Read the relevant docs before coding:

- `docs/plans/cl_mycelium_revenue_integrated_plan_v3.md`
- `docs/prompts/cl_mycelium_revenue_codex_prompt_pack_v3.md`
- organism docs under `docs/core/`
- current organism modules under `modules/organism/`
- current organism tests under `tests/test_organism_*.py`

## Safety requirements

Before changing behavior, check:

- Does this affect hints?
- Does this affect M2?
- Does this affect executor-visible decisions?
- Does this change production defaults?
- Does this bypass `cl_revenue_ops`?
- Does this require new operator consent?

If yes, add tests and explain clearly.

## Required tests

When touching organism compatibility or M2, run or update:

```bash
python -m pytest -q tests/test_organism_compat.py
python -m pytest -q tests/test_organism_m2_scope.py
python -m pytest -q tests/test_organism_acceptance_blocks_m2.py
python -m pytest -q tests/test_organism_m2_recovery_path_preservation.py
```

If a test is unavailable, explain why.

## Required report format

For every Codex task, report:

- files changed
- tests run
- whether executor boundary is preserved
- whether `["hive","hints"]` compatibility is preserved
- whether M2 scope is preserved
- whether production defaults changed
- whether any action/spend path changed
- follow-up risks

---
> Source: [lightning-goats/cl-mycelium](https://github.com/lightning-goats/cl-mycelium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
