## kozuchi-mini-swe-agent

> **Kozuchi Agent** — general-purpose coding agent implementation of Fujitsu Kozuchi Self-Evolved Agents.

# AGENTS.md — kozuchi-agent repository knowledge

## Project Overview

**Kozuchi Agent** — general-purpose coding agent implementation of Fujitsu Kozuchi Self-Evolved Agents.
Supports SWE-bench / HumanEval / LiveCodeBench from a single codebase.

- **Current score**: Qwen3.5-27B + TTS@8 — **74.80% SWE-bench Verified** (2026-03-26)
- Original codebase: `swe-sota-agent/` (minisweagent framework)

### Generalized Architecture (added 2026-04)

| Component | Responsibility |
|---|---|
| `benchmarks/base.py` | `BenchmarkAdapter` ABC — abstracts loading / environment / output format |
| `benchmarks/swe_bench.py` | Adapter for SWE-bench (reuses extra/swebench.py) |
| `benchmarks/code_gen.py` | Adapter for HumanEval / LiveCodeBench / generic code generation |
| `benchmarks/registry.py` | `get_adapter(benchmark_id)` — resolves Adapter from a string |
| `run_bench.py` | General-purpose batch runner (`kozuchi-run` command) |
| `configs/agent_codegen.yaml` | Single-phase config for code generation (SOLVE, `cat /solution.py`) |

---

## Directory Structure

```
kozuchi_agent/
├── agent.py           # KozuchiAgent / KozuchiAgentConfig
├── run_bench.py       # General-purpose multi-benchmark batch execution
├── benchmarks/
│   ├── base.py        # BenchmarkAdapter ABC
│   ├── swe_bench.py   # SWEBenchAdapter
│   ├── code_gen.py    # CodeGenAdapter (humaneval / livecodebench / codegen)
│   └── registry.py    # get_adapter(benchmark_id)
├── extra/
│   ├── swebench.py        # SWE-bench batch execution helpers (ProgressTrackingAgent)
│   ├── swebench_single.py # Single-instance execution
│   ├── cache_sifs.py      # SIF cache utility
│   └── utils/
│       └── batch_progress.py  # Rich progress bar manager
└── tools/             # Shell tools used by the agent
configs/
├── agent.yaml          # SWE-bench multi-phase config (production)
├── agent_merge.yaml    # Merge-mode variant
└── agent_codegen.yaml       # Single-phase config for code generation (HumanEval etc.)
```

---

## Key Classes

| Class | File | Description |
|---|---|---|
| `KozuchiAgent` | `agent.py` | Core agent (based on minisweagent) |
| `KozuchiAgentConfig` | `agent.py` | Config with added `submission_command` field |
| `BenchmarkAdapter` | `benchmarks/base.py` | ABC for benchmark-specific logic |
| `SWEBenchAdapter` | `benchmarks/swe_bench.py` | SWE-bench Docker environment + git diff |
| `CodeGenAdapter` | `benchmarks/code_gen.py` | Python Docker environment + /solution.py |
| `ProgressTrackingAgent` | `extra/swebench.py` | DefaultAgent subclass (progress tracking) |

---

## Entry Points

```bash
# General-purpose multi-benchmark batch execution
kozuchi-run --benchmark swe-bench-verified --source verified \
    --split test --slice 0:500 --workers 8

kozuchi-run --benchmark humaneval --config configs/agent_codegen.yaml

kozuchi-run --benchmark livecodebench \
    --source livecodebench/code_generation_lite --split test
```

## submission_command Design

`KozuchiAgentConfig.submission_command` (default: `"git add -A && git diff --cached"`)
controls the command executed at final-phase completion.

- SWE-bench: `git add -A && git diff --cached` → agent submits the patch
- HumanEval/LiveCodeBench: `cat /solution.py` → agent submits the written code

`run_bench.py` automatically injects `adapter.submission_command` into `config["agent"]["submission_command"]`.

---

## Development Setup

```bash
cd /path/to/kozuchi-agent
uv sync
uv run kozuchi-run --help
```

**Environment variables (for batch execution)**:

| Variable | Description |
|---|---|
| `MODEL_ID` | HF tokenizer identifier (required for token estimation) |
| `MODELS_DIR` | Local model weights root; `MODEL_ID` resolved under this dir |
| `VLLM_API_BASE` | vLLM server URL |
| `STEP_LIMIT` | Step limit (overrides config) |
| `MAX_PROMPT_TOKENS` | Prompt token limit |
| `INSTANCE_TIMEOUT` | Instance timeout (`MM:SS` format) |
| `ANTHROPIC_API_KEY` | **Required for Claude models** — used by Count Tokens API |
| `ANTHROPIC_BASE_URL` | Override Anthropic endpoint (default: `https://api.anthropic.com`) |

### Tokenizer Backends (`_init_tokenizer`)

| `_tokenizer_kind` | Trigger | Notes |
|---|---|---|
| `anthropic_count_tokens` | `_is_claude_model()` → `claude-*` / `anthropic/claude-*` | Calls `POST /v1/messages/count_tokens`; needs `ANTHROPIC_API_KEY` |
| `tiktoken` | `_is_openai_model()` → `openai/*`, `gpt-*`, `o1-*`, `o3-*`, `o4-*`, `chatgpt-*` | Offline BPE via `tiktoken`; `encoding_for_model()` with `o200k_base` fallback for unknown models |
| `hf` | `transformers.AutoTokenizer` loads from `MODELS_DIR` | Full chat template support |
| `tokenizers` | `tokenizers.Tokenizer.from_file(tokenizer.json)` fallback | Jinja2 template rendering |

Detection uses `query_model` (= `self.model.config.model_name` if available, else `MODEL_ID`), so `-m anthropic/...` works even when `MODEL_ID` points to a different model (e.g., Qwen).
**`MODEL_ID` is NOT required for Claude or OpenAI** — those backends only need `query_model` (from `-m` flag). Only local HF/tokenizers backends need `MODEL_ID` / `MODELS_DIR`. (BUG-010 fix, commit `74df408`)

**Claude**: `_count_tokens_anthropic()` — extracts system message, POSTs rest to API. No local file needed.  
**OpenAI**: `tiktoken.encoding_for_model(bare_model)` → falls back to `o200k_base` for unknown models (e.g., `gpt-5.2`). Token count uses OpenAI cookbook formula: 3 tokens/message overhead + content tokens + 3 reply-priming tokens.

---

## Related Repositories

- **swe-sota-agent**: `/path/to/swe-sota-agent/` — original agent implementation
- **kozuchi-bench**: `/path/to/kozuchi-bench/` — benchmark infrastructure (CLI/API/WebGUI)

---
> Source: [FujitsuResearch/kozuchi-mini-swe-agent](https://github.com/FujitsuResearch/kozuchi-mini-swe-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
