## biases

> - This repository studies bias and uncertainty in LLM-as-judge pairwise evaluation.

# AGENTS.md

## Project Context

- This repository studies bias and uncertainty in LLM-as-judge pairwise evaluation.
- The implemented bias families are position, authority, and bandwagon.
- Decoy artifacts are intentionally out of scope.
- Keep the repository infrastructure-neutral. Do not add machine-specific paths, node names, usernames, quotas, or cluster-specific troubleshooting notes to committed docs.
- Large datasets, model checkpoints, caches, logs, and outputs must stay outside Git. Use `BIASES_ARTIFACT_ROOT` to point runs at the active infrastructure's artifact volume.

## Development Rules

- Use Python 3.12.
- Prefer typed, small, pure functions for analysis code.
- Keep experiment configuration switchable through CLI arguments and environment variables.
- Do not hardcode model-cache paths, scheduler partitions, QOS names, or account names.
- Treat Slurm files as templates. Scheduler directives must be reviewed and adapted before submission on a new infrastructure.
- Do not commit generated outputs, downloaded datasets, checkpoints, local caches, or secrets.

## Verification

- Run `python3.12 -m py_compile` on changed Python files when full tests are not practical.
- Run targeted tests under `tests/` for changed modules when dependencies are available.
- For Slurm templates, run `bash -n` before submission.

---
> Source: [SuperCoolCucumber/biases](https://github.com/SuperCoolCucumber/biases) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
