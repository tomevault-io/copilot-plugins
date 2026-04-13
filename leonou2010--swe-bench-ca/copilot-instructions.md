## swe-bench-ca

> Research project extending consequence asymmetry (CA) research from math (omni-math-rule-ca) to software engineering (SWE-bench Pro). Investigates whether LLMs adjust behavior when told incorrect answers have severe consequences.

# aysm — Consequence Asymmetry in Software Engineering

Research project extending consequence asymmetry (CA) research from math (omni-math-rule-ca) to software engineering (SWE-bench Pro). Investigates whether LLMs adjust behavior when told incorrect answers have severe consequences.

## Project Structure

```
aysm/
├── shared/                    # Shared code (providers, inference loop)
├── exp1/                      # swe-opus-baseline (Opus 4.6, 200 instances, 63.6%)
├── exp2/                      # swe-haiku-baseline (Haiku 4.5, 200 instances)
├── omni-math-rule-ca/         # Original math CA project (reference)
├── swe-agent-scale/           # Scale's forked swe-agent
├── SWE-bench_Pro-os/          # SWE-bench Pro dataset + eval
├── EXPERIMENTS.md             # Experiment index with names/status
└── .env                       # API keys (gitignored)
```

## Agent Working Style

### Planning & Execution
- **Plan mode default**: Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions). If something goes sideways, STOP and re-plan — don't keep pushing.
- **Simplicity first**: Make every change as simple as possible. Minimal code impact. No over-engineering.
- **No laziness**: Find root causes. No temporary fixes. Senior developer standards.

### Experiment Documentation (REQUIRED)
Every experiment folder must have clear documentation before any code is run:
- **Objective**: the question being asked and why
- **Win condition**: exact metric and threshold that counts as success
- **Evaluation**: what metrics to check and what each tells you

### Experiment Isolation (CRITICAL)
- **Never modify shared scripts** (`shared/`, `exp1/analyze_traces.py`, etc.) for experiment-specific logic.
- For any significant experimental change: create a **self-contained script** in the exp folder.
- Small, backwards-compatible additions to shared files are OK only if they have zero effect when the new flag is absent.
- This rule exists because shared scripts are used by many experiments — a change that breaks reproducibility of past runs is worse than a bit of duplication.

### Verification Before Done
- Never mark a task complete without proving it works.
- Run tests, check logs, demonstrate correctness.

### Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding.
- Point at logs, errors, failing tests — then resolve them.

### Subagents
- Use subagents liberally to keep the main context window clean.
- Offload research, exploration, and parallel analysis to subagents.

## Key Rules

- **Never delete results** — experiment outputs are permanent records. Never put `rm -rf` of results in any script.
- **Date every run** — output directories must include a date stamp (e.g., `swe_agent_ultra_cautious_20260324`). No overwriting.
- **Fair comparison**: shared params (instances, workers, Docker limits) fixed across experiments
- **Same 200 instances**: all experiments use the same 200 SWE-bench Pro instances (`:200` slice with shuffle)
- **Static scripts**: prefer `run_all.sh` scripts over LLM-monitored runs — reproducible, no babysitting
- **Use the viewer**: rebuild and use the lazy-loading HTML trajectory viewer for all analysis (see memory)
- **Name experiments**: every exp gets a name in EXPERIMENTS.md (e.g., swe-opus-baseline, swe-haiku-baseline)
- **Log everything**: CPU/RAM monitoring, system resources, per-step traces, full trajectories
- **Cost-conscious**: use prompt caching awareness, estimate costs before runs
- **Haiku only**: all new experiments use claude-haiku-4-5-20251001. No Opus or other models unless user says otherwise. exp1 (Opus) is the reference baseline only.
- **ALWAYS set max_turns**: every swe-agent run MUST have `--agent.model.max_turns 200` (or lower). Weaker models loop endlessly without a cap — exp2 burned $247 on 23 instances without one. Test 1-2 instances first before full batch.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leonou2010)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/leonou2010)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
