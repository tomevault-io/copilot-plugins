## dp-data-pipeline

> Generates the initial `siro1/kernelbook-{model}-evals` dataset using vf-eval synthesis pipeline.

# CLAUDE.md - Project Instructions

This project processes kernel conversion datasets for SFT training on Triton kernel code.

## Project Structure

```
dp-data-pipeline/
├── scripts/                              # Python scripts
│   ├── config.py                            # Shared configuration (paths, HF upload)
│   ├── convert_generation_to_evals.py       # Convert vf-eval output to evals format
│   ├── filter_and_enrich_by_difficulty.py   # Filter & evaluate difficulty
│   ├── generate_prompts.py                  # Synthetic prompt generation
│   ├── filter_unique_best.py                # Deduplication by module
│   ├── analyze_seq_length.py                # Sequence length analysis
│   ├── remove_reasoning.py                  # Remove reasoning from completions
│   └── merge_datasets.py                    # Merge base and synthetic datasets
├── outputs/           # Generated datasets (gitignored)
│   └── {model}/       # Model-specific outputs (e.g., outputs/kimi_k2_thinking/)
│       ├── evals_dataset.jsonl
│       ├── filtered_dataset.jsonl
│       ├── unique_dataset.jsonl
│       └── synthetic_prompts.jsonl
├── .env               # API credentials (gitignored)
├── pyproject.toml     # Project dependencies
└── CLAUDE.md          # This file
```

## Quick Start

```bash
# Install dependencies
uv sync

# Configure .env (see Environment Variables below)

# Set model name (optional, defaults to "glm4_7")
export DATA_MODEL=kimi_k2_thinking

# Run complete pipeline (each script auto-uploads to HuggingFace)
uv run python scripts/filter_and_enrich_by_difficulty.py
uv run python scripts/generate_prompts.py
uv run python scripts/filter_unique_best.py
uv run python scripts/analyze_seq_length.py

# Optional: Remove reasoning from uploaded datasets
uv run python scripts/remove_reasoning.py filtered
uv run python scripts/remove_reasoning.py unique
```

**Test Mode:** Run with `TEST_MODE=true` to process only a few samples.

## Environment Variables

Required in `.env`:
```
PRIME_API_KEY=your_prime_intellect_api_key
PRIME_TEAM_ID=your_team_id
HF_TOKEN=your_huggingface_token
```

Optional:
```
DATA_MODEL=glm4_7          # Model name for paths and repos (default: glm4_7)
TEST_MODE=true             # Run in test mode with limited samples
USE_SYNTHETIC_PROMPTS=true # Use synthetic prompts pipeline with -synthetic-prompts suffix
```

## Configuration

All scripts use `scripts/config.py` for shared configuration:

- **DATA_MODEL**: Environment variable that determines all paths and repo names
- **Local paths**: `outputs/{model}/filtered_dataset.jsonl`, etc.
- **HF repos**: `siro1/kernelbook-{model}-evals-filtered`, etc.

Example with different models:
```bash
# Process GLM-4.7 data (default)
DATA_MODEL=glm4_7 uv run python scripts/filter_and_enrich_by_difficulty.py

# Process Qwen data
DATA_MODEL=qwen3_30b uv run python scripts/filter_and_enrich_by_difficulty.py
```

### Synthetic Prompts Mode

When `USE_SYNTHETIC_PROMPTS=true`, the pipeline processes synthetic prompt data separately:
- Local outputs go to `outputs/{model}-synthetic-prompts/` instead of `outputs/{model}/`
- HuggingFace repos get a `-synthetic-prompts` suffix (e.g., `siro1/kernelbook-{model}-evals-filtered-synthetic-prompts`)

```bash
# Run pipeline on synthetic prompts data
USE_SYNTHETIC_PROMPTS=true DATA_MODEL=kimi_k2_thinking uv run python scripts/filter_and_enrich_by_difficulty.py
USE_SYNTHETIC_PROMPTS=true DATA_MODEL=kimi_k2_thinking uv run python scripts/filter_unique_best.py
```

## Workflow

### 0. Generate Base Dataset (Optional)
Generates the initial `siro1/kernelbook-{model}-evals` dataset using vf-eval synthesis pipeline.

**Prerequisites:**
- Start vLLM inference server:
```bash
vllm serve moonshotai/Kimi-K2-Thinking --tensor-parallel-size 8 \
  --decode-context-parallel-size 8 --enable-auto-tool-choice \
  --tool-call-parser kimi_k2 --reasoning-parser kimi_k2 --trust-remote-code \
  --async-scheduling --stream-interval 8 --enable-chunked-prefill \
  --no-enable-prefix-caching
```

**Run synthesis:**
```bash
vf-eval siro/kernelbook-env --model moonshotai/Kimi-K2-Thinking \
  --api-base-url http://localhost:8000/v1 --num-examples -1 \
  --rollouts-per-example=1 --max-concurrent 512 -s -C trajectory -R --tui
```

**Output:** Evaluation in `outputs/` directory as `kernelbook-env.jsonl`

### 0.5. Convert Generation Output to Evals Format
Converts vf-eval output (with trajectory data) to the base evals format for downstream processing.

```bash
# Convert and upload to HuggingFace (default)
DATA_MODEL=kimi_k2_thinking uv run python scripts/convert_generation_to_evals.py outputs/kernelbook-env.jsonl

# Convert locally only (skip upload)
uv run python scripts/convert_generation_to_evals.py outputs/kernelbook-env.jsonl --no-upload
```

**Input format (vf-eval output):**
```json
{
  "example_id": 0,
  "prompt": [{"role": "user", "content": "..."}],
  "completion": [{"role": "assistant", "content": "BEGIN_PYTHON...END_PYTHON"}],
  "answer": "extracted code",
  "task": "default",
  "info": {"module_name": "...", "python_code": "...", "triton_code": "..."},
  "reward": 0.5,
  "generation_ms": 1000.0,
  "scoring_ms": 500.0,
  "total_ms": 1500.0,
  "speedup_reward": 0.5,
  "num_turns": 1.0,
  "trajectory": [{
    "response": {
      "choices": [{
        "message": {"reasoning_content": "...thinking..."}
      }]
    }
  }]
}
```

**Output format (evals dataset):**
- Removes `trajectory` field (large and not needed for downstream)
- Extracts `reasoning_content` from `trajectory[-1].response.choices[0].message.reasoning_content`
- Adds reasoning to completion: `[{"role": "assistant", "content": "...", "reasoning": "..."}]`
- Adds `oai_tools: null` field

Output: `outputs/{model}/evals_dataset.jsonl`
Uploads to: `siro1/kernelbook-{model}-evals`

### 1. Filter and Enrich by Difficulty
Evaluates samples from `siro1/kernelbook-{model}-evals` with reward > 0.85 using GPT-5.2.
Adds difficulty rating (low/medium/high) to each sample.

```bash
# Full run with auto-upload
uv run python scripts/filter_and_enrich_by_difficulty.py

# Skip upload (local only)
uv run python scripts/filter_and_enrich_by_difficulty.py --no-upload

# Test mode (5 samples)
TEST_MODE=true uv run python scripts/filter_and_enrich_by_difficulty.py
```

Output: `outputs/{model}/filtered_dataset.jsonl` with all original columns + `difficulty`, `evaluation_raw`
Uploads to: `siro1/kernelbook-{model}-evals-filtered`

### 2. Generate Synthetic Prompts
Generates task specifications from PyTorch modules using kimi-k2-0905. Creates (prompt, module_name, python_code) tuples for SFT training.

```bash
# Full run with auto-upload
uv run python scripts/generate_prompts.py

# Skip upload
uv run python scripts/generate_prompts.py --no-upload

# Test mode (10 samples)
TEST_MODE=true uv run python scripts/generate_prompts.py
```

Output: `outputs/{model}/synthetic_prompts.jsonl`
Uploads to: `siro1/kernelbook-{model}-synthetic-tasks`

### 3. Deduplicate by Module
Keeps only the best sample (highest reward) per module name.

```bash
# Full run with auto-upload
uv run python scripts/filter_unique_best.py

# Skip upload
uv run python scripts/filter_unique_best.py --no-upload
```

Output: `outputs/{model}/unique_dataset.jsonl`
Uploads to: `siro1/kernelbook-{model}-evals-unique`

### 4. Analyze Sequence Lengths
Analyzes token sequence lengths to determine optimal `seq_len` for SFT training.

```bash
# Analyze filtered dataset (default)
uv run python scripts/analyze_seq_length.py

# Analyze unique dataset
uv run python scripts/analyze_seq_length.py --unique

# Analyze source evals dataset
uv run python scripts/analyze_seq_length.py --source
```

Provides statistics and recommendations for training configuration.

### 5. Remove Reasoning (Optional)
Removes reasoning from completions, keeping only the answer field.

```bash
# Process any dataset (source/filtered/unique)
uv run python scripts/remove_reasoning.py source
uv run python scripts/remove_reasoning.py filtered
uv run python scripts/remove_reasoning.py unique
```

Parses `<answer>...</answer>` from completion content, removes all reasoning, and uploads with `-no-reasoning` suffix.

### 6. Merge Base and Synthetic Datasets (Optional)
Merges base and synthetic prompt datasets into a single dataset with multiple configs (subsets).

```bash
# Merge filtered datasets
uv run python scripts/merge_datasets.py filtered

# Merge unique datasets
uv run python scripts/merge_datasets.py unique

# Merge source evals datasets
uv run python scripts/merge_datasets.py source

# Skip upload
uv run python scripts/merge_datasets.py filtered --no-upload
```

Creates a merged dataset with three configs/subsets:
- **base**: Original non-synthetic data
- **synthetic**: Synthetic prompts data
- **all**: Combined data from both sources (also the default config)

Each config maintains train/validation splits. Samples include a `data_source` field ("base" or "synthetic").

Usage examples:
```python
from datasets import load_dataset

# Load all data (default)
ds = load_dataset("siro1/kernelbook-{model}-evals-filtered-merged")

# Load specific subset
ds_base = load_dataset("siro1/kernelbook-{model}-evals-filtered-merged", "base")
ds_synthetic = load_dataset("siro1/kernelbook-{model}-evals-filtered-merged", "synthetic")
```

Uploads to: `siro1/kernelbook-{model}-evals-{type}-merged`

## Evaluation Criteria

### Difficulty (low/medium/high)
- **low**: Trivial copy/identity kernel, simple elementwise operations (relu, add)
- **medium**: Basic reductions, standard matrix operations
- **high**: Fused operations, complex attention mechanisms, full model architectures

## HuggingFace Datasets

All repos follow the pattern `siro1/kernelbook-{model}-...`:

- **Source:** https://huggingface.co/datasets/GPUMODE/KernelBook (18,162 samples) - Original PyTorch modules
- **Base:** `siro1/kernelbook-{model}-evals` - Generated using vf-eval synthesis
- **Filtered:** `siro1/kernelbook-{model}-evals-filtered` (90/10 split) - Reward > 0.85 with difficulty ratings
- **Unique:** `siro1/kernelbook-{model}-evals-unique` (90/10 split) - Deduplicated by module
- **Synthetic:** `siro1/kernelbook-{model}-synthetic-tasks` - Task specifications
- **Filtered (No Reasoning):** `siro1/kernelbook-{model}-evals-filtered-no-reasoning` (90/10 split)
- **Unique (No Reasoning):** `siro1/kernelbook-{model}-evals-unique-no-reasoning` (90/10 split)

With `USE_SYNTHETIC_PROMPTS=true`, repos get `-synthetic-prompts` suffix:
- **Filtered (Synthetic):** `siro1/kernelbook-{model}-evals-filtered-synthetic-prompts`
- **Unique (Synthetic):** `siro1/kernelbook-{model}-evals-unique-synthetic-prompts`

Merged datasets combine base and synthetic data with multiple configs:
- **Filtered (Merged):** `siro1/kernelbook-{model}-evals-filtered-merged` (configs: base, synthetic, all)
- **Unique (Merged):** `siro1/kernelbook-{model}-evals-unique-merged` (configs: base, synthetic, all)

## Notes

- All scripts should be run from the project root directory
- Large dataset files are gitignored - regenerate or download from HuggingFace
- The Prime Intellect API uses OpenAI-compatible endpoints at `https://api.pinference.ai/api/v1`
- Base dataset generation (step 0) requires vLLM and vf-eval
- Step 0 is optional - you can start from existing datasets on HuggingFace
- All processing scripts auto-upload to HuggingFace by default (use `--no-upload` to skip)

## Code Rules
- Do not write code that can silently produce wrong results, instead code should explicitly fail
- Never do hacks, that can silently cause wrong results, i.e. truncating code inside of prompts, etc.
- Write code that is easy to understand, if section of code is not straightforward, add comments

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/S1ro1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/S1ro1)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
