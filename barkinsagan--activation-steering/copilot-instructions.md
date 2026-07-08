## activation-steering

> Experiments measuring how difference-in-means steering vectors affect LLM behavior across layers, using single-token and continuation scoring.

# Steering Vector Research Project

Experiments measuring how difference-in-means steering vectors affect LLM behavior across layers, using single-token and continuation scoring.

## Tech Stack
- Language: Python 3.x
- Key libraries: PyTorch, transformers, pandas, numpy, matplotlib, yaml
- Environment: venv / conda
- Models: HuggingFace models (e.g. meta-llama/Meta-Llama-3-8B)
- Data storage: local

## Project Structure
- /experiments — experiment infrastructure (runner.py, config.py, registry.py)
- /configs — YAML experiment configs (one file per experiment run)
- /analysis — post-hoc analysis scripts (compare.py)
- /data/eval — evaluation datasets (CSV: prompt, target [, false1, false2, false3])
- /data/prompts — steering prompts (pos.txt, neg.txt — one per line)
- /results — output CSVs, plots, config snapshots (one subdir per experiment_id)
- /docs — research log, findings, article drafts (SHARED WITH CLAUDE.AI)
- dim.py — DifferenceInMeansSteering implementation
- hook.py — ModelWithHooks (activation capture and patching)
- token_completion_test.py — continuation scoring sweep
- single_token_completion_test.py — single-token scoring sweep
- visualize_token_results.py / single_token_visualization.py — plot generation

## Commands
- `python experiments/runner.py configs/my_experiment.yaml` — run experiment
- `python experiments/runner.py configs/*.yaml` — run multiple experiments sequentially
- `python visualize_token_results.py` — generate continuation plots
- `python single_token_visualization.py` — generate single-token plots
- `python analysis/compare.py` — compare across experiments

## Conventions
- All experiments get a unique ID: exp_YYYYMMDD_shortname (set as `experiment_id` in the YAML config)
- Each experiment config lives at configs/{experiment_id}.yaml
- Results go in results/{experiment_id}/ (single_token/ and/or continuation/ subdirs)
- Every experiment must have a logged entry in docs/research-log.md
- Use seed=42 for reproducibility unless testing variance
- Plots: save as .png, always include axis labels and titles
- Resume mode: set `sweep.resume: true` in config to skip already-completed layers

## Experiment Workflow
When I ask you to run an experiment:
1. Create a unique experiment ID (exp_YYYYMMDD_shortname)
2. Write or copy a YAML config to configs/{experiment_id}.yaml
3. Run it: `python experiments/runner.py configs/{experiment_id}.yaml`
4. Capture results and auto-generate an entry in docs/research-log.md:

### exp_YYYYMMDD_shortname
**Date:** YYYY-MM-DD
**Goal:** What we were testing
**Method:** What we did (brief)
**Config:** configs/exp_YYYYMMDD_shortname.yaml
**Results:** Key numbers (accuracy delta, best layer, best coef, etc.)
**Plots:** results/exp_YYYYMMDD_shortname/...
**Verdict:** What this means / what to try next
**Status:** [success / partial / failed / inconclusive]

5. Update docs/experiment-index.md with a one-line summary
6. If the result is significant, add it to docs/findings.md

## Writing Bridge
When I say "prep for writing" or "sync for article":
1. Update docs/research-log.md with any unlogged experiments
2. Update docs/findings.md with latest key results
3. Generate a docs/writing-brief.md that summarizes:
   - What we've proven so far
   - Key numbers and results (best layers, coefs, accuracy deltas)
   - What plots are available
   - Open questions remaining
   - Suggested narrative arc for an article
4. Remind me to upload docs/ to my claude.ai Project

## Important Rules
- NEVER delete or overwrite previous experiment logs — append only
- Always save config snapshots alongside results (runner does this automatically)
- When results are surprising, re-run with a different random seed before concluding
- Commit after every successful experiment: `docs: log {experiment_id}`

---
> Source: [barkinsagan/Activation-Steering](https://github.com/barkinsagan/Activation-Steering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
