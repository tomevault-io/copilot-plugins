## metal-arm-harness

> This repository lets an agent drive a real robot arm safely from the shell.

# metal-arm-harness

This repository lets an agent drive a real robot arm safely from the shell.
There is no model API in the loop: **you are the policy.**

- To operate the arm, follow `docs/OPERATING.md` (procedure, bench facts,
  camera reading, past mistakes) — or invoke the `metal-arm-pick-place` skill.
- Motion always goes through `metal-arm-harness op …` against a running
  `metal-arm-harness serve`; never write to the CAN bus directly.
- Tests: `.venv/bin/python -m pytest tests` (no hardware needed).
- Do not commit, push, or open a PR without the user's explicit confirmation.

---
> Source: [makermods-robotics/metal-arm-harness](https://github.com/makermods-robotics/metal-arm-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
