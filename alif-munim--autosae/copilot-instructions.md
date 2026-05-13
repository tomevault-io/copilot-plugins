## autosae

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Does

Trains Sparse Autoencoders (SAEs) on neural network activations for mechanistic interpretability. SAEs decompose a frozen LLM's internal activations into sparse, interpretable features using dictionary learning. The LLM is never modified — the SAE is a separate model that learns to explain what the LLM does internally.

## Commands

```bash
# Install
pip install dictionary-learning
# or from source:
pip install poetry && poetry install

# Unit tests (no GPU required, runs in CI)
poetry run pytest tests/unit

# End-to-end tests (requires GPU + ~2 min on RTX 3090)
# These train SAEs on Pythia-70m and compare against expected metrics
poetry run pytest tests/test_end_to_end.py

# Single test
poetry run pytest tests/unit/test_dictionary.py::test_simple_autoencoder
```

## Architecture

### Training Pipeline

The training flow is: **text data → ActivationBuffer → trainSAE() → saved SAE**

1. **Data**: A string iterator (typically from `utils.hf_dataset_to_generator` streaming a HuggingFace dataset)
2. **`ActivationBuffer`** (`buffer.py`): Wraps an `nnsight.LanguageModel`. Runs LM forward passes to collect activations from a target submodule, stores them in a buffer, yields batches. Auto-refills when half-depleted. This is the main training bottleneck — the LM forward pass dominates.
3. **`trainSAE()`** (`training.py`): Main entry point. Takes a buffer + list of trainer configs. Handles the training loop, logging, checkpointing, optional wandb, and activation normalization.
4. **Trainers** (`trainers/`): Each SAE architecture has a paired trainer implementing its specific loss function and update step. All inherit from `SAETrainer` in `trainers/trainer.py`. The trainer's `update(step, activations)` method is the core training step.

### Key Abstractions

- **`Dictionary`** (ABC in `dictionary.py`): All SAE architectures implement `encode(x)`, `decode(f)`, and `forward(x, output_features=False)`. Concrete classes: `AutoEncoder`, `GatedAutoEncoder`, `JumpReluAutoEncoder`, `AutoEncoderTopK`, `BatchTopKSAE`, etc.
- **`SAETrainer`** (`trainers/trainer.py`): Base class. Subclasses implement `update()` and `loss()`. Also defines `ConstrainedAdam` optimizer that constrains decoder weights to unit norm.
- **`ActivationBuffer`** (`buffer.py`): Three variants exist — `ActivationBuffer` (main, text-based), `NNsightActivationBuffer` (token-based), and `HeadActivationBuffer` (per-attention-head, Llama-specific).

### Trainer → Dictionary Mapping

| Trainer | Dictionary Class | Key Hyperparameter |
|---|---|---|
| `StandardTrainer` | `AutoEncoder` | `l1_penalty` |
| `GatedSAETrainer` | `GatedAutoEncoder` | `l1_penalty` |
| `TopKTrainer` | `AutoEncoderTopK` | `k` (active features) |
| `BatchTopKSAETrainer` | `BatchTopKSAE` | `k` |
| `JumpReluTrainer` | `JumpReluAutoEncoder` | `bandwidth` |
| `PAnnealTrainer` | `AutoEncoderNew` | `p_start`, `p_end` |

### Evaluation

`evaluation.py` computes: L2 loss, L1 loss, L0 (active features), variance explained, cosine similarity, L2 ratio, relative reconstruction bias, fraction alive, and **loss recovered** (% of model CE loss preserved when swapping in SAE reconstruction).

### Model Compatibility

Works with any HuggingFace model via `nnsight`. For models that `nnsight` can't load from string (e.g., VLMs like Qwen3.5), load with `AutoModelForCausalLM.from_pretrained()` then wrap: `LanguageModel(torch_model, tokenizer=tokenizer)`.

`utils.py:get_submodule()` has explicit support for: `GPTNeoXForCausalLM`, `Gemma2ForCausalLM`, `Qwen2ForCausalLM`, `Qwen3ForCausalLM`, `LlamaForCausalLM`.

### Qwen-Specific Handling

`ActivationBuffer` has two Qwen-specific parameters:
- `remove_bos=True`: Qwen has no BOS token; this removes the first non-pad token instead
- `max_activation_norm_multiple=10`: Filters high-norm activation sinks that hurt SAE training

### Key Training Options

- `normalize_activations=True` in `trainSAE()`: Normalizes activations to unit mean squared norm. Weights are scaled back before saving so inference doesn't need normalization. Critical for hyperparameter transfer across layers/models.
- `autocast_dtype=t.bfloat16`: Significant speedup with minimal quality impact.
- `lr=None` in TopK trainer configs: Auto-selects LR using `2e-4 / sqrt(dict_size / 2^14)` scaling.
- Typical dict_size: 8x-16x the activation dimension.

## Dependencies

Core: `nnsight>=0.3.0,<0.4.0`, `torch`, `transformers`, `datasets`, `wandb`, `einops`. Managed via Poetry (`pyproject.toml`). Python 3.10+.

## Detailed Documentation (docs/)

The `docs/` directory contains detailed notes organized by topic:

- **Architecture**: [`docs/architecture/overview.md`](docs/architecture/overview.md) — training pipeline, supported architectures, evaluation metrics. [`docs/architecture/model-compatibility.md`](docs/architecture/model-compatibility.md) — nnsight workarounds, VLM loading, Qwen-specific handling.
- **Training**: [`docs/training/hyperparameters.md`](docs/training/hyperparameters.md) — current BatchTopK configs (exp=2 candidate, exp=3 committed), cross-layer performance, key insights. [`docs/training/multi-gpu.md`](docs/training/multi-gpu.md) — parallel layer training strategy. [`docs/training/train-qwen35-script.md`](docs/training/train-qwen35-script.md) — CLI reference for `train_qwen35.py`.
- **Benchmarks**: [`docs/benchmarks/model-comparison.md`](docs/benchmarks/model-comparison.md) — training speed comparison across Qwen3.5-0.8B, RecurrentGemma-2B, Phi-4-mini.
- **Environment**: [`docs/environment/setup.md`](docs/environment/setup.md) — machine specs, package versions, dependency upgrade history, libstdc++ fix.

## Autoresearch (mar10 session)

- **Experiment log**: [`docs/experiments/README.md`](docs/experiments/README.md) — master narrative of all experiments
- **Experiment reports**:
  - [`docs/experiments/001-baseline.md`](docs/experiments/001-baseline.md) — baseline TopK SAE, identified dead features as #1 problem
  - [`docs/experiments/002-dead-threshold-and-sweep.md`](docs/experiments/002-dead-threshold-and-sweep.md) — lowered dead threshold + auxk/expansion/k sweep; expansion=8 won
  - [`docs/experiments/003-expansion-k-sweep.md`](docs/experiments/003-expansion-k-sweep.md) — expansion 4/6/8 × k 48/64/96; expansion=4 breakthrough (alive=0.547)
  - [`docs/experiments/004-lr-expansion-warmdown-sweep.md`](docs/experiments/004-lr-expansion-warmdown-sweep.md) — LR=2x auto achieves recovered=0.858
  - [`docs/experiments/005-fine-lr-and-k-sweep.md`](docs/experiments/005-fine-lr-and-k-sweep.md) — diminishing returns from hyperparameter tuning
  - [`docs/experiments/006-architecture-experiments.md`](docs/experiments/006-architecture-experiments.md) — BatchTopK breakthrough (alive=0.916), k-annealing/resampling failed
  - [`docs/experiments/007-batchtopk-sweep.md`](docs/experiments/007-batchtopk-sweep.md) — BatchTopK k=80 meets ALL targets (alive=0.868, recovered=0.874)
  - [`docs/experiments/008-cross-layer-validation.md`](docs/experiments/008-cross-layer-validation.md) — config generalizes across all 8 layers
  - [`docs/experiments/009-batchtopk-expansion-k-auxk.md`](docs/experiments/009-batchtopk-expansion-k-auxk.md) — larger expansion hurts alive; k=88 alt config
  - [`docs/experiments/010-optimizer-schedule.md`](docs/experiments/010-optimizer-schedule.md) — warmdown=0.3 breakthrough (recovered 0.874→0.908)
  - [`docs/experiments/011-warmdown-cosine-sweep.md`](docs/experiments/011-warmdown-cosine-sweep.md) — warmdown 0.25-0.35 optimal, cosine LR promising
  - [`docs/experiments/012-noise-and-k-sweep.md`](docs/experiments/012-noise-and-k-sweep.md) — noise: recovered ±0.023, alive ±0.034
  - [`docs/experiments/013-training-technique-sweep.md`](docs/experiments/013-training-technique-sweep.md) — no_grad_proj, exp3/5, auxk variations
  - [`docs/experiments/014-no-grad-proj-validation.md`](docs/experiments/014-no-grad-proj-validation.md) — no_grad_proj within noise
  - [`docs/experiments/015-cross-layer-validation-v2.md`](docs/experiments/015-cross-layer-validation-v2.md) — warmdown=0.3 validated, layer 4: 0.622→0.845
  - [`docs/experiments/016-layer-specific-tuning.md`](docs/experiments/016-layer-specific-tuning.md) — late layers var problem fundamental
  - [`docs/experiments/017-jumprelu-architecture.md`](docs/experiments/017-jumprelu-architecture.md) — JumpReLU far worse than BatchTopK
- **Research agenda**: [`docs/research-agenda.md`](docs/research-agenda.md) — prioritized ideas and next steps
- **Literature**: [`docs/literature/arxiv-notes.md`](docs/literature/arxiv-notes.md) — paper notes with provenance (search → experiment mapping)
- **Known issues**: [`docs/known-issues.md`](docs/known-issues.md) — bugs, artifacts, gotchas
- **Failed ideas**: [`docs/ideas/failed.md`](docs/ideas/failed.md) — dead ends with reasons (do not retry)

---
> Source: [alif-munim/autosae](https://github.com/alif-munim/autosae) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
