## symbolic-transformer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Digital Transformer — a symbolic next-token predictor that replaces floating-point weights with hierarchical label pattern matching. Uses a Claude-powered debug agent to iteratively improve the model via tool-use.

## Commands

```bash
# Dependencies
uv sync

# Tests (158 tests)
uv run pytest
uv run pytest tests/test_transformer.py                          # single file
uv run pytest tests/test_mlp.py::TestMLP::test_basic_fire -v     # single test

# Training
uv run python train.py --iterations 10 --dry-run                 # no API calls
uv run python train.py --iterations 10 --wandb --name "exp-001"  # real training
uv run python train.py --checkpoint checkpoints/model.json       # resume

# Benchmark (stdlib call tracer)
uv run python benchmark.py --wandb
```

## Architecture

### Data flow
```
Text → GPT-2 tokenizer → EmbeddingTable → [Attention → MLP] × 4 layers → UnembeddingHead → softmax → loss
```

Each token position carries a `TokenState` (positive + negative label sets) that flows through layers. Labels live in a hierarchy DAG — if token has "dog" and hierarchy says dog→mammal→animal, the token implicitly has "mammal" and "animal".

### Key modules (digital_transformer/)

- **routing_dag.py** — `LabelHierarchy` (DAG with bloom-filter-accelerated walks), `TokenState`, `OverflowPolicy`. The core data structure everything else builds on.
- **counting_bloom.py** — `CountingBloomFilter` with 4-bit counters, supports add/remove. Used for fast ancestor lookups.
- **dot_product.py** — `LabelMatrix`, `Neuron`. Pattern matching via inverted index + bloom-guided batched walk. Score = count of matched pattern labels (+1 positive, -1 negative).
- **mlp.py** — `MLP`, `GatedMLP`. Returns `(NeuronOutput delta, fired_list)` — does NOT modify the token. Caller applies skip connection via `merge_output()`. Neurons fire in ascending margin-above-threshold order (strongest last wins conflicts).
- **attention.py** — Multi-head. Each `AttentionHead` has own query MLP + value MLP + k. Query uses DELTA labels only (not full token state). Keys annotated with relative position labels (rel:-1..1024, buckets, binary bits). Position hierarchy set up by `ensure_position_hierarchy()`.
- **unembed.py** — `UnembeddingHead`. Scores × `logit_scale` (default 5.0) then softmax. Score is based on pattern labels, not token state size.
- **transformer.py** — `DigitalTransformer`, `EmbeddingTable`, `TransformerBlock`, `StepTrace`, `LayerCheckpoint`. Orchestrates the pipeline. `forward_batch()` for training, `forward()` for inference.
- **debug_agent.py** — Claude-powered agent with tools: `inspect`, `query`, `simulate`, `mutate`. Two-phase loop: work (mutate model) then reflect (see eval results). System prompt contains full architecture docs. Uses Anthropic streaming API with prompt caching.
- **checkpoint.py** — JSON save/load of full model state including multi-head attention.

### Critical semantics

- **Hierarchy walks go UP** (child→parent). Token has "dog", pattern has "mammal" → match. Reverse does NOT match.
- **MLP returns delta, not mutated token.** The transformer block applies `merge_output(token, delta, hierarchy)` for the skip connection.
- **Attention query uses delta only.** Labels the query MLP *added*, not the full token state. Prevents shared categories from drowning the signal.
- **Unembed logit_scale=5.0** means 1 extra matching label = 5 logit points. Very impactful.
- **Neuron labels must be unique** within a LabelMatrix. Enforced at add time.

### Training loop (train.py)

1. Load TinyStories (HuggingFace streaming), split 80/20 train/eval
2. Build model with category hierarchy (punctuation, letter/uppercase/lowercase, digit, whitespace)
3. Each iteration: forward_batch → agent inspects/mutates → eval on held-out data → agent reflects
4. Agent feedback passed to next iteration (up to 32k chars)
5. Checkpoints saved as JSON, agent logs to `logs/agent/iter_NNNN.json`

### Debug agent tools

4 tools with unified interfaces:
- **inspect(id)** — Objects: `model`, `block:L`, `comp:L:mlp`, `cp:pos:L:sublayer`, `n:L:comp:idx`, `label:name`, `unembed:token`, `sim:id`
- **query(type)** — `list_components` (with depth), `connection`, `search_neurons`, `list_neurons`, `list_labels`, `top_losses`, `loss_by_label`
- **simulate(positive, run_through)** — Test tokens through mlp/attention/unembed/full_block
- **mutate(type, params)** — `add_label`, `add_edge`, `add_embedding`, `add_neuron`, `add_attention_head`, `add_query_neuron`, `add_value_neuron`, `update_unembed`, `set_threshold`, `batch_edges`, `batch_labels`

## Conventions

- Model name for API: `claude-sonnet-4-20250514` (supports prompt caching, unlike `claude-sonnet-4-6`)
- Neuron labels: short, descriptive, unique. E.g. `mlp/L0/tag_verb`, `query/L1/find_subject`
- Agent logs: `logs/agent/iter_NNNN.json` — full conversation + tool calls
- wandb project: `digital-transformer-train`, categories: `train/`, `eval/`, `graph/`, `neurons/`, `agent/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ueaj-Kerman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
