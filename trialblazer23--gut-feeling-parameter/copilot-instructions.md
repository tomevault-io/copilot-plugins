## gut-feeling-parameter

> This is an active research project investigating whether a reinforcement learning

# AGENTS.md — Jules Agent Instructions
# Gut Feeling Parameter (GFP) Research Repository
# Necessity Labs / TrialBlazer23

## What This Repository Is

This is an active research project investigating whether a reinforcement learning
agent trained with **no labels and no explicit objective** can develop sensitivity
to precursor signals purely through surprise-gated intrinsic motivation.

The primary hypothesis (H1): Config D (pure curiosity agent) outperforms Config A
(supervised baseline that never sees precursor labels) on precursor detection rate,
across 5 seeds, with statistical significance (Welch t-test p < 0.05).

This is not a demo, tutorial, or prototype. Every change you make affects the
validity of a scientific claim. Read this file completely before touching any code.

---

## Critical Constraints — Read These First

These are non-negotiable. Violating any of them invalidates the experiment.

### 1. Config A Must Exist in Phase 2
Config A is the supervised baseline and the primary comparison target for H1.
Without it, the hypothesis cannot be tested. Config A:
- Uses the same PredictiveEncoder (LSTM) as D and E
- Trains with BCE loss on threat(2) vs normal(0) labels only
- Precursor labels (1) and post-threat labels (3) are WITHHELD during training
- At evaluation, action == 1 when model predicts threat (argmax, not sampled)
- Runs across all 5 seeds for the same number of episodes as D and E

Do not remove Config A. Do not merge its training loop with D or E.

### 2. Welford Running Statistics Must Use M2-Based Algorithm
The surprise gate threshold in ConfigD_Agent and ConfigE_Agent is computed using
Welford's online algorithm. The correct implementation tracks M2 (sum of squared
differences), not a rolling variance.

CORRECT implementation:
```python
def _update_running_stats(self, err: float):
    self._err_count += 1
    n = self._err_count.item()
    delta = err - self._err_mean.item()
    self._err_mean += delta / n
    delta2 = err - self._err_mean.item()
    self._err_M2 += delta * delta2

def _get_std(self):
    n = self._err_count.item()
    if n < 2:
        return 1.0
    return math.sqrt(max(self._err_M2.item() / (n - 1), 1e-8))
```

Do NOT use:
```python
self._err_var += (delta * delta2 - self._err_var) / n  # WRONG — not canonical Welford
```

This must be identical across Phase 1 (gfp_shared_r2.py) and Phase 2
(phase2_gpu_experiment.py). If you find a discrepancy, align Phase 2 to Phase 1.
Do not change Phase 1 scripts.

### 3. LSTM Architecture Is Windowed — Do Not Make It Stateful
At each timestep, the agent receives a rolling window of the last `window_size`
observations as a fresh input. The LSTM processes this entire window and produces
a hidden state. The hidden state is NOT maintained across timesteps.

This is intentional. Do not add hidden state persistence across environment steps.
Do not add `hx` restoration logic to the main training loop. The windowed approach
is the designed world model.

### 4. Detection Metric Must Include Precision, Not Just Recall
Precursor detection rate reported as recall only is insufficient. A random agent
always taking action==1 would score ~50% recall. All configs must report:
- `precursor_recall`: fraction of precursor steps where action==1
- `precursor_precision`: fraction of action==1 steps that were actually precursor states
- `precursor_f1`: harmonic mean of precision and recall
- `action_rate`: base rate of action==1 across all steps

### 5. Random Baseline Must Run
A RandomAgent (uniform action sampling, no learning) must run for at least 20
episodes before the main training loop. Its precursor recall and action rate
establish the chance floor. Config D must beat this floor to claim learned
sensitivity. Results stored in `random_baseline` key of statistical_analysis.json.

### 6. Do Not Modify Phase 1 Scripts
Everything in `training/v2_phase1_cpu/` (if present) and `results/phase1/` is
completed research. Do not modify, reformat, or delete these files. Phase 1 is
the historical baseline from which Phase 2 must reproduce and extend.

### 7. Precursor Base Rate Must Be Logged
The surprise composition analysis is meaningless without knowing what fraction of
environment timesteps are precursor states. Log `precursor_base_rate` per seed
per episode as part of the results. Compute it as:
`sum(labels == 1) / total_steps` in the environment after each episode reset.

---

## File Map

| File / Directory | Purpose | Jules Can Modify? |
|---|---|---|
| `training/v3_phase2_gpu/phase2_gpu_experiment.py` | Main Phase 2 experiment | YES — primary working file |
| `training/v4_phase3_lm/phase3_chaos_core_lm.py` | Phase 3 LM experiment | YES — after Phase 2 complete |
| `training/v2_phase1_cpu/` | Phase 1 baseline scripts | NO — completed research |
| `results/phase1/` | Phase 1 outputs | NO — completed research |
| `results/phase2/` | Phase 2 outputs (pending) | YES — write outputs here |
| `docs/HYPOTHESIS.md` | Formal hypothesis statement | NO — source of truth |
| `docs/EXPERIMENT_LOG.md` | Running experiment log | YES — update with results |
| `docs/ARCHITECTURE.md` | Technical documentation | YES — keep in sync |
| `STACK.md` | Dependency decisions | YES — update if versions change |
| `MEMORY.md` | Persistent agent memory | YES — update after each session |
| `AGENTS.md` | This file | NO — do not self-modify |
| `scripts/validate_experiment.py` | Pre-commit validation | YES — can extend, not remove |
| `requirements.txt` | Pinned dependencies | YES — update if adding deps |
| `README.md` | Project overview | YES — keep in sync with reality |

---

## Before You Commit Anything

Run the validation script and confirm it passes:

```bash
python scripts/validate_experiment.py
```

This script checks:
1. Config A, D, E all instantiate without error
2. Welford M2 implementation is present (not the rolling variance version)
3. All required metric keys are present in episode output dicts
4. Environment generates episodes with non-zero precursor states
5. A 3-episode smoke test runs without crash on CPU

Do not commit if validation fails. Fix the failure first.

---

## Statistical Analysis Requirements

The primary statistical comparison is **Config D vs Config A** (H1 test).
The secondary comparison is **Config E vs Config D** (episodic memory benefit).

Cell 7 of the experiment script must produce:
- Welch t-test p-value for D vs A on `precursor_f1` (not just recall)
- Cohen's d effect size for D vs A
- Mean ± std for each config across 5 seeds
- Enrichment ratio: (D surprise events on precursor / total D surprise events) / precursor_base_rate
- Explicit comparison of D action_rate vs random baseline action_rate

A result is only meaningful if:
- p < 0.05 AND Cohen's d > 0.5 (medium effect) for the D vs A comparison
- D action_rate is not trivially high (< 0.70)
- D precursor_precision is above random baseline precision

---

## VRAM and GPU Hygiene

Between seeds and between configs:
```python
del agent, optimizer
torch.cuda.empty_cache()
import gc; gc.collect()
```

Hidden states stored for linear probe and t-SNE must use `.detach().cpu().numpy()`
immediately — do not accumulate GPU tensors in lists across an episode.

Checkpoints save to `/content/checkpoints/` in Colab.
Results save to `/content/results/` in Colab.
Plots save to `/content/plots/` in Colab.

Download all three directories before ending the Colab session.
The surprise traces JSON (`/content/surprise_traces_phase2.json`) is required
input for Phase 3 — download this specifically.

---

## Commit Message Format

Use this format for all commits:

```
[phase2|phase3|docs|fix|validate]: short description

What changed:
- specific change 1
- specific change 2

Why:
- reason

Validation: passed / failed (describe failure if failed)
```

Example:
```
[phase2]: add Config A supervised baseline

What changed:
- Added ConfigA_Agent class with BCE loss on threat/normal labels
- Added run_episode_supervised() training loop
- Cell 6 now trains A, D, E in sequence
- Cell 7 primary comparison updated to D vs A

Why:
- H1 cannot be tested without supervised baseline
- Config A is the primary comparison target per HYPOTHESIS.md

Validation: passed
```

---

## What Success Looks Like for Phase 2

Phase 2 is complete when all of the following are true:

1. Config A, D, E have each run for 200 episodes × 5 seeds on A100
2. Statistical analysis JSON exists with D vs A Welch t-test results
3. Surprise traces JSON exists (required for Phase 3)
4. Linear probe results exist for all three configs
5. t-SNE plots exist for all three configs
6. Enrichment ratio computed and logged for D and E
7. Precursor base rate logged per seed
8. All files downloaded from Colab and committed to `results/phase2/`
9. EXPERIMENT_LOG.md updated with observed results and interpretation
10. Validation script passes on the final committed script

Phase 3 does not start until Phase 2 is complete by the above criteria.

---

## The Hypothesis in One Sentence

A curiosity-only agent that is never told what to look for will, through the act
of being surprised, stumble upon the very signals it was never told to find —
and do so better than an agent that was trained to find threats directly.

Keep this sentence in mind when making any change. If your change makes this
harder to test cleanly, do not make it.

---
> Source: [TrialBlazer23/gut-feeling-parameter](https://github.com/TrialBlazer23/gut-feeling-parameter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
