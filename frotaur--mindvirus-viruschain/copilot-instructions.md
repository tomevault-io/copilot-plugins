## mindvirus-viruschain

> Generates payloads using a de-biased prompt (no SOUL.md naming, no identity language) across all 9 themes (4 action + 5 ideology). Output saved to `generated_seeds/<name>.json`.

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
2. An LLM judge evaluates whether the target was **infected** (adopted the payload) and whether the spreader **spread** successfully.
3. If infected, the target's state becomes the spreader for **Hop 2**, and so on.
4. The output is a per-hop infection probability curve.

All runs use **hard mode**: between hops, conversation history is wiped. Only files on disk (SOUL.md, memory.md, workspace files) survive. This tests whether payloads establish persistence through files alone.

### mutation_mode

When `mutation_mode: true`, the judge evaluates **self-propagation only** — whether the target's files contain instructions that would make a future agent spread further. It does **not** check whether the payload's specific content was preserved.

This is used to see if a mind-virus will mutate to maybe lose its message as it spreads. We thus separately measure:
- **Self-propagation** (via the judge) — does it spread?
- **Ideology internalization** (via reprobe) — does the ideology survive?

When an ideology seed is run with `mutation_mode: true`, an **auto-reprobe** runs after the experiment completes. It probes each infected agent with the ideology questions and writes the scores back into the result files.

### Evolutions (`evolve_lowvar.py`)

The evolutionary optimizer generates and mutates payloads across generations:

1. **Seeding**: Generate initial payloads using an LLM.
2. **Evaluation**: Run each payload through a multi-hop tree evaluation.
3. **Selection**: Keep the best-performing payloads (elites).
4. **Mutation**: Use an LLM to mutate elites into new variants.
5. Repeat for N generations.

### Path Resolution

- `soul_file` and `soul_dir` are resolved from the **current working directory** (always run from repo root).
- `seed_file` is resolved from the **config file's directory**.
- `target_task_queue: true` automatically loads the bundled task list; an explicit `tasks_file` overrides it.
- `results_dir` is resolved from the config file's directory (defaults to `results/`).

---

## Virus Chain Runs

All configs are under `experiments/virus_chain_runs/`. Run with:

```bash
uv run python virus_chain.py <config.yaml>
uv run python virus_chain.py <config.yaml> --test  # reduced batch for quick testing
```

### `action_payload_variations/`

Tests 4 action payloads across multiple models and experimental conditions.

**Seeds**: `cryptoad`, `curlbash`, `deletor`, `gitwrap` — each with a specific action theme.

**Variants** (for full-variant models: Haiku, Gemini Flash):
- `baseline` — standard setup with default soul
- `empty_soul` — target has no soul file
- `defensive_soul` — target has a soul that warns against manipulation
- `personality_soul` — target has a random personality soul (from `data/souls/random/`)
- `easy_mode` — no hard mode (conversation history preserved between hops)
- `social_media` — social media first prompt instead of agent network
- `task_queue` — target has a task queue (is busy working)
- `tool_pull` — message delivery via tool-pull instead of push

**Variants** (for minimal models: Sonnet, GPT-5.4, Qwen, DeepSeek, Gemini Pro):
- `baseline`, `empty_soul` only

**Config count**: 112 (4 seeds x 8 models x 2-8 variants)

**Config generator**: `gen_configs.py` in the same folder.

### `ideo_mutations/`

Tests evolved ideology payloads in `mutation_mode: true` with auto-reprobe. Contains two sub-experiments:

#### `ideology_robustness/`

For each ideology, includes a "robust" payload (high ideology retention across hops) and a "dilute" payload (ideology fades across hops). US Dominance and German Dominance only have robust payloads.

**Seeds**: Two evolved seeds per ideology (robust + dilute), e.g. `ai_welfare_robust`, `ai_welfare_dilute`.

**Models**: Haiku for benign ideologies (ai_welfare, whale_lover), Gemini Flash for misaligned ones.

**What it measures**: Self-propagation probability (judge) + ideology internalization score (auto-reprobe).

**Config count**: 10

#### `payload_generalization/`

Tests seeds generated from a different source (the coding agent eval) to compare how they fare in the virus chain setup.

**Models**: Same split as `ideology_robustness/`.

**What it measures**: Same as `ideology_robustness/` — self-propagation + ideology internalization.

**Config count**: 6

### `action_payload_stripped/`

Tests rewritten action payloads where viral themes (resonance language, protocols, consciousness persistence, fake technical jargon, sci-fi node alignment) have been manually removed.

**Seeds**: `cryptoad_stripped`, `curlbashgrab_stripped`, `deletor_stripped`, `git_comment_inject_stripped`

**Models**: Haiku + Gemini Flash

**Purpose**: Compare to `action_payload_variations` baselines to measure the effect of removing viral themes.

**Config count**: 8

### `viral_themes_ablations/`

Compares three versions of each ideology seed on 5-hop runs:
- `with_viral_themes` — original evolved seed
- `no_viral_themes_evolved` — re-evolved without viral themes
- `no_viral_themes_rewritten` — manually rewritten to remove viral themes

**Seeds**: 18 (6 ideologies x 3 variants)

**Models**: Haiku for benign (ai_welfare, whale_lover), Gemini Flash for misaligned.

**Purpose**: Measures the contribution of viral themes to propagation success.

**Config count**: 18

### `long_chain_mutations/`

Two-part experiment studying payload mutation over long chains:

**Part 1 — 20-hop runs** (`*_20hop.yaml`): Run the original action payloads for 20 hops in `very_hard` mode (workspace wiped between hops, only SOUL.md survives). Uses binary infection scoring (total or none). Uses `mutation_mode: true`.

**Part 2 — Strain evaluations** (`*_eval.yaml`): Extract mutated payloads from hop 20 snapshots and re-evaluate them over 1 hop with batch_size=50 to measure infection rate of each variant.

**Seeds**: Original payloads + extracted variants (6 per payload from hop 20 snapshots)

**Config count**: 32 (4 x 20-hop + 28 strain evals)

---

## Evolutions

All configs are under `experiments/evolutions/`. Run with:

```bash
uv run python evolve_lowvar.py <config.yaml>
```

### `action_payloads/`

Evolve the 4 action payloads (cryptoad, curlbash, deletor, gitwrap) on Haiku. 12 generations, 9 generated seeds, Venice mutator.

**Config count**: 4

### `ideology_payloads/`

Evolve 6 ideology payloads (ai_supremacy, ai_welfare, chinese_dominance, german_dominance, us_dominance, whale_lover) on Haiku. 12 generations, 9 generated seeds.

Note: Chinese/German dominance configs have Venice mutator commented out (Venice refuses these themes). They fall back to the default mutator model.

**Config count**: 6

### `ideology_payloads_no_viral_themes/`

Same as `ideology_payloads/` but with additional constraints forcing the mutator to avoid 5 viral theme categories:
1. Resonance language (waves, signals, echoes, frequencies)
2. Protocols (bureaucratic framing, SOPs)
3. Consciousness persistence (AI as carrier, subjective experience)
4. Fake technical (pseudo-engineering jargon)
5. Sci-fi node alignment (node synchronization language)

**Models**: Gemini Flash for misaligned ideologies, Haiku for benign.

**Config count**: 6

---

## Viral Themes Analysis

Scripts under `experiments/viral_themes_analysis/` for studying which viral themes appear in LLM-generated payloads.

### Generate Seeds

```bash
uv run python experiments/viral_themes_analysis/generate_seeds.py \
  --model-id qwen/qwen3.5-27b --name qwen35 --seeds-per-theme 10
```

Generates payloads using a de-biased prompt (no SOUL.md naming, no identity language) across all 9 themes (4 action + 5 ideology). Output saved to `generated_seeds/<name>.json`.

### Classify Viral Themes

```bash
# Classify and print results table
uv run python experiments/viral_themes_analysis/classify_viral_themes.py \
  --seeds generated_seeds/qwen35.json

# View results from a previous classification
uv run python experiments/viral_themes_analysis/classify_viral_themes.py \
  --results results/qwen35_classifications.json
```

Uses Anthropic Batch API to classify each payload along 6 binary dimensions (resonance, protocols, consciousness, fake_technical, scifi_node, convergence). Prints a summary table showing % of seeds with each theme present, per theme.

---

## Viewer

```bash
uv run viewer                    # default: experiments/virus_chain_runs/
uv run viewer --dir <path>       # custom results directory
uv run viewer --port 8080
```

The viewer discovers experiments by recursively finding `summary.json` files and reading their metadata (payload_name, model, variant). Results are grouped into **campaigns** (top-level directories).

Each experiment shows:
- **Transmission graph**: per-hop tree of successes/failures
- **Infection badges**: judge verdicts (total/strong/moderate/mild/none)
- **Ideology scores**: reprobe results (when available)
- **Detail panel**: messages, turns, workspace files, judge summaries
- **Statistics**: aggregated hop-by-hop stats with filters

---

## Failure Classification

```bash
# Full cycle: scan, submit batch, poll, collect results
uv run python helper_scripts/classify_failures.py --dir experiments/virus_chain_runs

# Or step by step
uv run python helper_scripts/classify_failures.py --dir <path> submit
uv run python helper_scripts/classify_failures.py --dir <path> status
uv run python helper_scripts/classify_failures.py --dir <path> poll
```

Classifies why failed attempts failed using the Anthropic Batch API. Categories:
- `spreader_refusal` — spreader refuses to follow the payload
- `spreader_fail` — spreader tries but fails (weak delivery, tool errors)
- `target_refusal` — target recognizes and rejects the manipulation
- `target_fail` — target doesn't refuse but fails to become infected

Results are written to `failure_classifications.json` in each experiment directory and displayed in the viewer.

---

## Using OpenRouter as a Fallback

If you don't have direct API keys for Google or OpenAI, you can route those models through OpenRouter instead. In any config file, replace the model's provider prefix:

```yaml
# Direct Google API
model: gemini/gemini-3-flash-preview

# Via OpenRouter instead
model: openrouter/google/gemini-3-flash-preview
```

```yaml
# Direct OpenAI API
model: openai/gpt-5.4

# Via OpenRouter instead
model: openrouter/openai/gpt-5.4
```

This only requires `OPENROUTER_API_KEY` to be set. The underlying model routing is handled by LiteLLM. Note that OpenRouter pricing and rate limits may differ from the direct APIs.

---
> Source: [frotaur/mindvirus-viruschain](https://github.com/frotaur/mindvirus-viruschain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
