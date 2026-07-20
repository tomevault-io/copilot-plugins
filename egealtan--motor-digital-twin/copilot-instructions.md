## motor-digital-twin

> Guidance for Claude Code when working in this repo.

# CLAUDE.md

Guidance for Claude Code when working in this repo.

## Current state (handoff)

This is where the project is. A fresh session on any machine can pick up from here. Read
`docs/twin-spec.md` and `docs/twin-explained.md` first; they are the locked design.

What works. The twin core is built and wired into the autoresearch loop. The tuning baseline, the
twin encoder, the bits-per-spike metric, the noise-ceiling trust metrics, and the two trust gates are
in place and pass on MC_Maze_Small val. With the strengthened baseline (features velocity, speed,
acceleration, position; glm_alpha 1e-3) the velocity-tuning anchor reaches median cc_abs 0.413. The
bidirectional GRU twin (d_model 64) reaches cc_abs 0.667 with 95 percent of neurons above the 0.4
gate and bps 0.233 at a 60 second budget, so `autotwin gate` passes both gates. The base twin config
is in `configs/base.yaml`.

The metric to optimize. Match the vision digital twins: Poisson loss to fit, correlation-to-average
(and its noise-ceiling-normalized form, feve) to judge, with a 0.4 per-neuron keep gate. feve needs
repeated reaches to estimate the noise ceiling, which MC_Maze_Small val does not have, so on small
data bps is a stable proxy to confirm the machinery. On the full dataset feve is the real objective.

The full dataset is now the base config. `configs/base.yaml` points at MC_Maze full (DANDI 000128,
about 2295 trials over 108 conditions with many repeats each), `task.py` optimizes `feve`, and the
objective is averaged over condition-stratified cross-validation folds (`cv_folds=3`). feve needs
condition repeats in the eval set, which random folds destroy, so `prepare_cv_folds` stratifies
WITHIN each condition: it round-robins every condition's repeats across the folds, so each fold's
eval holds a share of all 108 conditions (5 to 9 repeats each) and feve is computable per fold while
the split rotates. `configs/experiments/twin_full.yaml` and `tuning_full.yaml` remain for a one-off
`train_twin.py --config ...` baseline-vs-twin check.

The metric had a bug, now fixed (read this). The first feve compared single-trial predictions to a
trial-averaged noise ceiling and dropped the /R on the noise floor. Because the twin predicts from
single-trial hand movement, its per-trial prediction captures within-condition variance the ceiling
counted as noise, so feve ran past 1 (test feve 1.098) and swung about 0.06 with the seed. `_ceiling`
now computes the standard Schoppe/Cadena feve at the PSTH level: average the prediction to a PSTH,
noise floor = across-repeat variance / R, feve = 1 - (mse(observed_psth, model_psth) - floor)/signal.
A perfect model gives 1 (synthetic: 1.107 at R=8 to 1.002 at R=100), an imperfect model gives below
1, and cc_norm = sqrt(feve), so cc_norm squared equals feve.

Verified results under the corrected metric. The best config under 3-fold condition-stratified CV
scores val feve 0.924 (fold spread 0.017), cc_abs 0.875, bps 0.353. The same config at seed 1 gives
feve 0.905, cc_abs 0.873, so the seed gap shrank from 0.06 to 0.019 (the residual is finite-sample
bias at 5 to 7 repeats, not a bug). On the held-out test split, scored once off-search: feve 0.952,
cc_abs 0.885, bps 0.358, all neurons above the 0.4 gate. Test slightly beats val on every metric, so
the twin generalizes with no overfitting. This is a strong, trustworthy motor cortex digital twin.

The twin has largely converged (cc_abs about 0.88, near the ceiling). feve is now trustworthy but
still carries small finite-sample noise at this repeat count, while cc_abs and bps are the most
seed-stable, so watch all three. Do not overindex on the test number: it was read once and must not
become a selection target.

Why one worker on the M1: two workers time-slice the one GPU and give each experiment a different
step count under the fixed wall-clock budget, which confounds the objective. On a multi-GPU box this
does not apply.

What has not happened yet. The long search. The movement-captioning experiment (search the movement
that most drives a well-predicted neuron, then describe it in language) is later work, not this phase.

The current run. The search runs on the M1 with one worker on the CV objective:
`autotwin loop --proposer llm --budget 500 --duration-s 43200` (12 hours), after `autotwin seed`.
Each experiment is about 7 to 8 minutes now (3 folds). It is operator-invoked. Stop it any time with
`autotwin stop`, which writes `journal/STOP`; the worker finishes its current experiment and exits.
On a GPU box the same command runs, and `autotwin sweep --workers 2` is fine there.

Launch it detached, or it dies with the session. When an automated session (like this one) starts
the run, a normal background job gets killed when the harness reaps it. Launch through
`scripts/launch_detached.py` (a new-session `Popen`, since macOS has no `setsid`), for example
`python scripts/launch_detached.py /tmp/sweep.log caffeinate -is uv run --extra nlb autotwin loop
--proposer llm --budget 500 --duration-s 43200`. `caffeinate` keeps the Mac awake. A person running
it in their own terminal does not need this.

Which model actually proposed. The llm proposer records the model that truly answered each call, not
just the one requested. Guardrails can reroute (for example claude-fable-5 served by
claude-opus-4-8); the journal stores `requested_model` and `served_model` per proposal, the live
watch flags a reroute, and `autotwin usage` prints the served-model breakdown and reroute count.

Watch the run live. Point a Monitor at `scripts/watch_journal.py`. It prints one line per experiment
with the metrics, which model answered, and the hypothesis, without touching the journal. feve is the
search target. Check that cc_abs and bps move with feve; feve up while cc_abs or bps falls means the
twin is fitting spike-rate noise, which is the sign to steer.

This repo is public. Keep it that way: no personal links, no private paths, and no session links in
commit messages.

## What this is

This is an exploratory solo project. It fits a motor cortex digital twin, a model that maps hand
movement to the firing rate of every recorded neuron, and uses an autoresearch loop to search for the
best twin. The dataset is NLB MC_Maze_Small. The first milestone is trust: a learned twin that beats a
linear velocity-tuning baseline on held-out trials, reported with a noise-ceiling-normalized metric.
The movement-captioning goal is why the project exists, but it is later work.

Start here, in order: `docs/twin-spec.md` for the locked build spec, `docs/twin-explained.md` for the
step-by-step explanation, `docs/models.md` for the two models, `README.md` for the full picture,
`docs/DESIGN.md` for the reference numbers, and `program.md` for the steering document the language
model reads.

Before touching the data or the models, look at the data with `uv run autotwin viz-data`. It plots
the movement, the true PSTH, and a baseline prediction.

## Commands

Everything runs through uv.

- `uv sync` installs the core stack. `uv sync --extra nlb` adds the real NLB data tools.
  `uv sync --extra dev` adds pytest and ruff.
- `uv run pytest` runs the tests. They use the synthetic backend and need no download.
- `uv run --extra dev ruff check --fix && uv run --extra dev ruff format` lints and formats.
- `uv run autotwin <command>` is the operator CLI: data, run, loop, sweep, stop, seed, usage,
  leaderboard, show, confirm, gate, plot, report, viz-data.

On an arm64 Mac, point uv at an arm64 Python (for example a Homebrew python3.13) so it does not pick
an x86 one. On a Linux GPU box, plain `uv sync` picks the right build.

## Conventions

- Python 3.10 or later. Use uv for venvs, dependencies, and running scripts.
- Write prose in the plain direct style: simple words, complete sentences, no dashes, no "we", no
  analogies, no filler. This applies to the README, docs, comments meant for people, and commit and
  PR messages. It does not apply to code.
- Add a dependency only when it is needed.
- Do not over-engineer. This is a research repo.

## Things to know

- The search is started by a person. Never start a long unattended sweep on your own. `loop` and
  `sweep` are operator-invoked.
- The benchmark is frozen. `task.py`, `data.py`, `twin.py`, and the frozen config keys are the
  benchmark. Proposers change only the mutable config.
- The metric is bps (encoding bits per spike over all neurons), maximize. The bits-per-spike math is
  the exact nlb_tools code, copied into `autoresearch/_nlb_eval.py`. Do not reimplement it.
- The twin core, the tuning baseline, and `score_twin` live in `autoresearch/twin.py`. The training
  entrypoint is `train_twin.py`. The runner invokes it (`runner.TRAIN_ENTRYPOINT`).
- The search objective is bps over `data.cv_folds` cross-validation folds (default 3). The folds have
  no condition structure, so the trust metrics (cc_abs, feve) come from single-split runs
  (`cv_folds=1`), which is what `gate` and `viz-data` use.
- Known issue in the twin core. `build_twin_arrays` passes NLB's `eval_cond_idx` straight through,
  but NLB stores it as one entry per condition (the trial indices in that condition), while
  `score_twin` needs a per-trial condition vector. So the single-split trust-metric path through
  `train_twin.py`, and `confirm`'s one-time test eval, currently crash on real NLB data. `gate` and
  `viz-data` route around this in `cli.py` (`_pertrial_cond_idx`) and work today. The fix is to
  normalize `eval_cond_idx` into a per-trial vector inside `build_twin_arrays`; the CV search path is
  unaffected because the folds carry no condition structure.
- nlb_tools pins an old pandas. A `[tool.uv]` override installs a current pandas on Python 3.12 and
  later, and `data.py` replaces one loader function at runtime so it still works.
- The test split's held-out spikes are withheld in the public data file, so the test path loads NLB's
  released targets (`eval_data_test.h5`, downloaded on first use).
- The language model proposer defaults to Fable 5 through the `claude` CLI, which uses the Claude
  subscription. Do not switch it to a metered API without asking.
- Keep every file command inside this repo. Do not run a search across the home folder, because it
  triggers macOS folder-access prompts.
- `autotwin gate` is the trust check. Run it after changes to the data, the twin, or the metric.

## Running overnight on a single GPU, for example a 4090 in WSL

The laptop 60 second budget is short. Longer training makes the twin steadier, so real runs go on a
GPU.

First, two things need you. Make sure `nvidia-smi` works in the WSL shell. Make sure the `claude` CLI
is installed in WSL and logged in, since the proposer uses it. Or set `ANTHROPIC_API_KEY` and add
`--extra llm`.

Then, in `configs/base.yaml`, set `train.time_budget_s` to about 150 and `train.device` to `auto`,
and run:

```bash
uv sync --extra nlb
uv run autotwin gate                      # confirm the twin still beats the baseline
uv run autotwin seed                      # put the base twin in the journal
uv run autotwin sweep --workers 2 --duration-s 28800 --proposer llm
```

These models are small, well under 1 GB of VRAM, and each experiment waits for its language model
proposal, during which the GPU is idle. Two workers overlap well on one GPU. Do not go higher than 2.

In the morning:

```bash
uv run autotwin confirm best --seeds 5    # re-check the best over seeds, score test once
uv run autotwin report                    # write journal/report.html
uv run autotwin usage
git add journal && git commit -m "record: overnight GPU sweep"
```

## Commit style

Conventional commits, for example `feat:`, `fix:`, `docs:`, `chore:`. Write the description in the
plain style. No dashes.

---
> Source: [egealtan/motor-digital-twin](https://github.com/egealtan/motor-digital-twin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
