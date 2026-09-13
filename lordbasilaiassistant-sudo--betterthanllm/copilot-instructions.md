## betterthanllm

> Instructions for coding agents working in this repository. Humans should read

# AGENTS.md

Instructions for coding agents working in this repository. Humans should read
[CONTRIBUTING.md](CONTRIBUTING.md) instead — it says the same things with more
context.

## Setup

```bash
pip install -r requirements.txt
```

Python 3.10+, numpy only. No GPU, no PyTorch, no network access needed.

## Verify before and after every change

```bash
python3 experiments/identity_tests/experiment_v4.py   # ~4s
python3 experiments/identity_tests/experiment_v5.py   # ~3s
python3 experiments/fitness/gauntlet.py               # ~6s, exits non-zero if fooled
```

`experiment_v4` and `experiment_v5` are **bit-identical across runs**. Diff the
output before and after your change. If any number moved, that is a behavioral
change and your PR must say which number, by how much, and why.

`gauntlet.py` runs four deliberately degenerate substrates as controls and
**exits 1 if it accepts any of them**. Treat a non-zero exit as a hard failure.

## Rules specific to this repo

1. **Never optimize an absolute similarity.** `signature_similarity` is a cosine
   over non-negative vectors with a floor near 0.64, and the headline metric is
   *maximized by a substrate that learns nothing* (`lr=0` scores 0.9137 vs the
   published 0.8051; a frozen `top_k=1` substrate scores 1.0000). Only the
   control-subtracted measures in `gauntlet.py` may be optimized.

2. **A margin is only comparable within one configuration.** Changing `top_k`,
   `plan_weight` or the learning rates moves the metric's own floor. Normalize
   by headroom (`headroom_norm` in `gauntlet.py`) before comparing across
   configs. A search already exploited this once — see the audit note in that
   function.

3. **Every claim needs its measurement; every measurement needs its control.**
   Do not write a number into a docstring, README or commit message that you did
   not just run. This repo has a commit correcting exactly that mistake.

4. **Do not add dependencies.** numpy-only is a deliberate property.

5. **Do not add claims about consciousness, sentience or AGI.** The repo
   measures behavior and explicitly takes no position on experience. See
   `llms.txt` for the list of things that are not claimed.

6. **Do not delete a result to make something pass.** If a number changes,
   explain it. The audit history (`AUDIT.md`,
   `experiments/wake_up_test_v3/FACT_CHECK_REPORT.md`) is the format: numbers
   stay, overclaims go.

## Where things live

| Path | What |
|---|---|
| `experiments/identity_tests/experiment_v4.py` | The substrate itself, plus tests T1-T7. Read this first. |
| `experiments/identity_tests/experiment_v5.py` | Tests T8-T10; imports from v4. |
| `experiments/fitness/gauntlet.py` | The un-gameable scorer and its controls. |
| `experiments/fitness/search.py` | Config search with a held-out seed family. |
| `experiments/wake_up_test_v1..v4/` | Historical experiments. Do not modify; they are a record. |
| `AUDIT.md`, `FINDINGS.md`, `results.json` | Claim surfaces. Keep them consistent with the code — a number appears in all three. |

## Commit style

Explain what was measured, not what was intended. Include the numbers. If you
fixed something, state what it was wrong by.

---
> Source: [lordbasilaiassistant-sudo/BetterThanLLM](https://github.com/lordbasilaiassistant-sudo/BetterThanLLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
