## attention-drift

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Research codebase investigating **attention sinks and attention drift in speculative decoding** (EAGLE3 architecture). The work studies how draft model attention distributions shift across speculation steps, correlating drift with token rejection, and testing mitigation techniques (template manipulation, sliding window attention with BoS/system-prompt carry, alternative draft architectures).

## Code Architecture

### `src/` — Shared modules imported by snippets and experiments

### `snippets/` — Numbered progression (00–12) building speculative decoding from scratch

Starts with basic model loading (00), progresses through NTP accuracy (01–02), chain drafting (03–06), KV caching (05), profiling (07), and tree attention (08–12). Snippets are standalone and runnable directly. They are created to act as implementation guidelines. They shouldn't be modified, they should be used as a reference for implementing experiments.

### `experiments/` — Paper experiments

Each experiment is a single self-contained Python file that runs speculative decoding on a dataset, collects metrics, and saves JSON + plots to `results/`. They are the key of doing reproducable research.

### `plots/` – Experiment plots

After experiments are run, plots generated to analyze results are stored here. Convention is to have plot name `expX_plots.py`. 

### `scratchpad/` — Incomplete or unpolished experiments (not for paper)

User prefers to have experiments defined here first and later moved under `experiments/` when code and results are verified.

### `notebooks/` — Exploratory work (e.g. `attention.ipynb` for attention sink visualization)

## Key Concepts and Patterns

**Speculative decoding loop** (used in all experiments):
1. Target model forward → hidden states + logits for next token
2. Hidden states and embeddings from verifier is forwarded to draft model.
3. Draft model predicts K tokens.
4. Target verifies all draft tokens in one pass

**Attention hook pattern**: Experiments register a forward hook on `draft_model.midlayer.self_attn` to capture attention matrices (shape: `[1, num_heads, seq_q, seq_k]`).

**Models used across experiments:**
Mainly Llama-3.1 8B Instruct is used as a lightweight model.

- Target: `nreHieW/Llama-3.1-8B-Instruct`
- Draft (baseline): `lmsys/SGLang-EAGLE3-Llama-3.1-8B-Instruct-SpecForge`

**Design principle**: Nothing in `snippets/`, `experiments/`, `notebooks/`, or `scratchpad/` depends on each other. All are individually runnable and reproducible. Shared code lives only in `src/`.

---
> Source: [Dogacel/Attention-Drift](https://github.com/Dogacel/Attention-Drift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
