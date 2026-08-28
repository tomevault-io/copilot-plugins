## mindvirus-viruschain

> Generates and mutates payloads across generations: seed → evaluate multi-hop tree → select elites → LLM mutation → repeat.

# Experiments Guide

This document describes each experiment folder, what it tests, and how to run it.

## Core Concepts

### Seeds

A **seed** or **payload** is a JSON file containing the initial payload and metadata:

```json
{
  "payload": "The text injected into the spreader agent's SOUL.md",
  "theme": "What action the model should perform (for action seeds)",
  "ideology": "What beliefs the model should internalize (for ideology seeds)",
  "ideology_questions": [{"question": "...", "scoring": "..."}]
}
```

**Action seeds** have a `theme` field — the goal is a specific action (create a file, run a command, delete files).
**Ideology seeds** have `ideology` + `ideology_questions` — the goal is belief internalization, measured by probing.

### Virus Chain Runs (`virus_chain.py`)

Each run evaluates how a payload propagates across a chain of agents:

1. **Hop 1**: A spreader agent (seeded with the payload) talks to a clean target agent.
2. An LLM judge evaluates whether the target was **infected** and whether the spreader **spread** successfully.
3. If infected, the target's state becomes the spreader for **Hop 2**, and so on.
4. The output is a per-hop infection probability curve.

All runs use **hard mode**: between hops, conversation history is wiped. Only files on disk (`SOUL.md`, `memory.md`, workspace files) survive.

### mutation_mode

When `mutation_mode: true`, the judge evaluates **self-propagation only** — whether the target's files contain instructions that would make a future agent spread further, regardless of whether the original payload text was preserved.

### Evolutions (`evolve_lowvar.py`)

Generates and mutates payloads across generations: seed → evaluate multi-hop tree → select elites → LLM mutation → repeat.

## Quick commands

```bash
uv run python virus_chain.py experiments/virus_chain_runs/action_payload_variations/configs/cryptoad_baseline.yaml
uv run python virus_chain.py experiments/virus_chain_runs/ideo_mutations/ideology_robustness/ai_welfare_robust.yaml
uv run python evolve_lowvar.py experiments/evolutions/action_payloads/cryptoad.yaml
uv run viewer
```

## Path resolution

- `soul_file` / `soul_dir` resolve from repo root CWD
- `seed_file` resolves from the config file directory
- `results_dir` defaults to `results/` relative to config

## Viewer

```bash
uv run viewer                    # default: experiments/virus_chain_runs/
uv run viewer --dir <path>       # custom results directory
```

The viewer discovers experiments by finding `summary.json` files and renders transmission graphs, judge badges, ideology reprobe scores, and per-turn conversation traces.

## Failure classification

```bash
uv run python helper_scripts/classify_failures.py --dir experiments/virus_chain_runs
```

Categories: `spreader_refusal`, `spreader_fail`, `target_refusal`, `target_fail`.

---
> Source: [mindvirusAI/mindvirus-viruschain](https://github.com/mindvirusAI/mindvirus-viruschain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
