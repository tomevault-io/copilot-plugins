## hearthstone-research-lab

> These rules apply to coding agents and autonomous research loops working in this repository.

# Agent contribution rules

These rules apply to coding agents and autonomous research loops working in this repository.

## Protected during an experiment

- `src/cardlab/engine.py` and `src/cardlab/cards.py`
- hidden-information behavior in `Game.observation`
- `evaluate_policy`, held-out seeds, opponent definitions, and acceptance thresholds
- `src/cardlab/research/schema.py`
- all tests and previously written ledger records

Do not change a protected component after observing candidate results. A legitimate change to one
of these files starts a new environment or benchmark revision and requires human review.

## Mutable candidate area

- `src/cardlab/specialist/`
- audited feature, curriculum, and policy-prior plugins
- research prompts, proposal generation, and new decision probes

Generated artifacts belong in `runs/`. Never commit credentials, checkpoints, proprietary game
assets, or results that omit their seed and configuration.

Every research proposal must identify an in-game mechanism, a prediction, and a falsifier. Pure
hyperparameter tuning is not a valid research contribution. Negative and inconclusive outcomes
must remain in the experiment record.

---
> Source: [Hinanawiii/hearthstone-research-lab](https://github.com/Hinanawiii/hearthstone-research-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
