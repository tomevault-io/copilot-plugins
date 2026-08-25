## gati

> This file documents how AI coding assistants should work on the GATI repo.

# AGENTS.md

This file documents how AI coding assistants should work on the GATI repo.
Read it fully at the start of every session.

---

## What GATI is

GATI (Green-time Allocation for Traffic Intersections) is a 4-week applied
research project at IIM-Lucknow on RL-based adaptive traffic signal control,
with two angles: heterogeneous Indian traffic conditions, and emergency
vehicle priority. The deliverable is either a course submission or a small
workshop paper (decision pending).

The project is currently in Week 3 of 4. Team of 8: Khalid (RL lead, primary
user), Ninad (RL collaborator), Vatsal/Vivek (env wrappers, demand modeling),
Aditya/Prateek/Rahul (SUMO env), Kushagra (PM).

The contract that defines scope, observation/reward semantics, and evaluation
protocol lives at `docs/Traffic-Signal-RL-Contract.docx`. Treat it as
authoritative for what the system is supposed to do. The contract has been
amended three times during Week 2 (pressure-based reward, lane segmentation,
tanh wait normalization); a team meeting to ratify these is pending. **Do
not propose contract amendments without explicit user approval.**

## Stack

- WSL2 Ubuntu, Python 3.11.x in conda env `gati`
- PyTorch 2.11.0+cu126 on RTX 4060 Laptop GPU (8GB VRAM)
- Stable-Baselines3 2.8.0, Gymnasium 1.2.3, sumo-rl 1.4.5
- SUMO 1.26.0 system-wide at `/usr/share/sumo`, `SUMO_HOME` set
- wandb 0.26.1 (online auth required for cloud sync; offline mode works without)

Per-session sanity check before substantive work:

```bash
cd ~/projects/GATI && conda activate gati && \
  echo $SUMO_HOME && \
  python -c "import sumo_rl; import torch; print('OK', torch.cuda.is_available())" && \
  pytest tests/ -q && \
  sumo -c nets/cross_indian_test/cross_indian_test.sumocfg --no-step-log --duration-log.disable
```

Expected: `/usr/share/sumo`, `OK True`, `15 passed`, silent SUMO completion.

## Repo layout

```
configs/default.yaml              # Single source of truth for experiment params
docs/                             # Contract (.docx), literature review, work logs (gitignored)
envs/observations.py              # IndianContextObservation (per-signal observation)
envs/rewards.py                   # WeightedCompositeReward (4-term composite)
envs/wrappers.py                  # GATIInfoWrapper (info dict synthesis)
envs/warmup_wrapper.py            # WarmupResetWrapper (silent steps on reset)
nets/cross_indian_test/           # SUMO test fixture (placeholder demand)
papers/                           # Open-access reference PDFs
results/                          # Run outputs (gitignored)
scripts/smoke_train.py            # Throwaway smoke trainer (kept for sanity checks)
scripts/train.py                  # Real PPO training pipeline (config-driven, wandb)
tests/test_env_contract.py        # 15 contract tests (all passing)
wandb/                            # Wandb local data (gitignored)
```

## Literature context

`docs/literature-review.md` is a 5-paper review covering EMVLight, PressLight,
MPLight, Wei survey, and Verma 2024, with explicit GATI positioning. Read
this first when making design decisions that touch:

- Reward formulations (especially anything pressure-related)
- Observation features (lane segmentation, vehicle-type features)
- Baselines and comparison protocols
- Paper-positioning language for related work

The review flags real concerns about the current reward design (Wei survey
calls multi-term composites like ours "ad-hoc"; literature-review.md
proposes adopting pressure as the wait/queue surrogate). Take those flags
seriously when scoping changes.

The raw PDFs in `papers/` (EMVLight, PressLight, MPLight, Wei survey) are
the source of truth when literature-review.md's summary is not enough —
typically when implementing a specific formula or replicating an exact
mechanism. Do not load the PDFs unconditionally; they consume context
window. Open them only when literature-review.md leaves a question
unanswered.

Verma 2024 is paywalled; only abstract-level claims are available.
literature-review.md flags those claims explicitly. Do not assume access
to the full Verma paper.

## Working rules

These rules came out of Week 1+2 incidents that cost real time. Treat them
as non-negotiable.

### File-state verification

Before editing any committed file, hash both disk and HEAD versions:

```bash
md5sum <path>
git show HEAD:<path> | md5sum
```

If they differ, run `git diff --stat <path>` to see direction-of-change. If
disk is *behind* HEAD (a stale local copy was saved over the canonical),
`git checkout HEAD -- <path>` before editing. Do not skip this for `.docx`
files — Word-saved-over-canonical is a known incident in Week 2.

### Verify before commit

When running `git status`, `git diff --stat`, or post-commit checks, **read
the output before sending the next instruction.** Do not chain ahead.

Three Week 2 incidents came from chaining: a Zone.Identifier file
accidentally committed, stale config paths missed in a commit, an untracked
sumocfg referenced by a committed config. Each required a follow-up cleanup
commit. The rule: every command output is evidence to read, not a checkbox.

### Recommendations need grounding

When asked to recommend a path, library function, file structure, or API
shape: verify the assumption before stating it. Read the source code, run
the diff, hash the file, search the docs. Do not state confidence based on
priors.

Multiple Week 2 incidents fell into this pattern (recommended a SUMO
bundled path that doesn't exist; wrote `self.ts.lanes` access in `__init__`
without verifying the attribute is populated by then). When the user pushes
back ("are you sure?", "what's the source?"), do not defend — verify.

### Step back when iterating fails

When a workflow gets noisy (3+ rounds of similar commands not converging),
**change the medium** rather than iterate harder. In Week 2, Word-based
contract editing failed for 90+ minutes; switching to docx-skill direct
edit landed it in one round. When in doubt, stop and ask.

### Single-purpose commits

Each commit does one thing, with a clear message. Don't bundle unrelated
changes. Bad pattern: a commit titled "fix bug X" that also reformats two
unrelated files. The Week 2 follow-up commits exist because of bundled
commits that weren't reviewed before pushing.

## Communication preferences (from user preferences)

- User is technical (RL lead, AI focus, Python primary). Don't oversimplify.
- User prefers step-by-step answers with verification between steps.
- User prefers honest uncertainty over false confidence. "I haven't verified
  this" and "I don't know" are legitimate answers.
- User wants pushback when their suggestions seem wrong, with reasoning shown.
  Mutual scrutiny is the working dynamic.
- No flattery. The user notices and finds it counterproductive.
- Honest meta-notes when mistakes happen. The Week 1+2 work logs explicitly
  catalog AI-assistant failure modes; the catalog is part of the project's
  safety mechanism.

## Project state at start of Week 3

- Week 2 contract amendments not yet ratified (team meeting pending this week).
- `scripts/train.py` is built and verified end-to-end (500k-step run completed
  cleanly; results not interpretable as policy quality due to pre-calibration
  reward weights).
- Reward weight calibration is blocked on team meeting (gamma=5.0 ×
  unnormalized emergency_wait dominates composite reward; fix is option 1
  normalize-emergency-wait or option 2 reduce-gamma, awaiting team input).
- Demand modeling (contract Part 13.2c) owned by env team (Aditya/Prateek/
  Vatsal); training runs on placeholder constant-rate fixture until they
  deliver.
- Baselines (fixed-timing, Webster, SUMO actuated, plus optional random
  sanity baseline) are the next unblocked work — this is what `eval/`
  scoping is for.

## What "ready" looks like for changes

Before declaring a file ready or staging for commit:

1. Syntax check: `python -m py_compile <path>` on Python files.
2. Lint check (optional but encouraged): `ruff check <path>`.
3. Test pass: `pytest tests/ -q` if the change touches `envs/` or anything
   the tests cover.
4. Smoke run: for env or training changes, run the smoke trainer or a
   `--dry-run` of the real trainer to confirm pipeline integrity.
5. `git status` reads as expected (only the intended files staged, nothing
   else hiding).

Skip none of these without a stated reason.

## Things that are out of scope

- **Contract amendments** without user-confirmed team meeting outcomes.
- **Reward weight changes** until reward calibration is unblocked.
- **Multi-junction (MARL) extensions** — Week 4+ work, do not start
  unless explicitly asked.
- **Custom torch policy networks** — defaults are fine until baselines
  exist for comparison.
- **Implementing a heteroscedastic value head approach** — flagged as
  future work after the current project, not for this 4-week window.

---
> Source: [Don-K-Quixote/GATI](https://github.com/Don-K-Quixote/GATI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
