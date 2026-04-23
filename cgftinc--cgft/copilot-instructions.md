## cgft

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install in development mode
pip install -e .[dev]

# Lint
ruff check src/

# Format
ruff format src/

# Type check
mypy src/

# Run tests
pytest

# Run a single test
pytest tests/path/to/test_file.py::test_function_name

# Add a dependency
uv add <package>
```

Line length is 100 characters. Ruff rules in use: E, F, I, N, W, UP.

**Python version:** 3.12 required. Python 3.13 is explicitly unsupported due to a `pathlib.Path` pickle incompatibility (enforced in `src/synthetic_data_prep/utils.py`).

## Architecture

This library provides an end-to-end pipeline for fine-tuning LLMs on RAG workloads using reinforcement learning. The pipeline flows: **document chunking → corpus indexing → synthetic QA generation → RL environment → training job**.

### Module Overview

**`chunkers/`** — Markdown document chunking with a 3-stage pipeline:
1. Split by markdown headers (H1/H2/H3), preserving hierarchy in metadata
2. Fuse adjacent short sections to avoid over-fragmentation
3. Recursive character splitting for large sections with overlap

`Chunk` is a frozen dataclass with auto-computed SHA256 hash. `ChunkCollection` provides file-structure-aware neighbor lookup and context retrieval.

**`corpus/`** — Swappable backend abstraction via the `ChunkSource` Protocol:
- `CorporaChunkSource` — CGFT Corpus API (BM25 search)
- `TpufChunkSource` — Turbopuffer vector DB (embeddings + BM25)

Both are used to index chunks and expose a search API consumed by RL environments.

**`qa_generation/`** — Synthetic QA dataset generation via `generate_dataset()`. Produces two types:
- *Single-hop*: one chunk answers the question (1 LLM call)
- *Multi-hop*: multiple chunks required (2 LLM calls)

Output is a `QADataset` of `QADataPoint` objects, serializable to JSONL or JSON.

**`envs/`** — RL training environments extending `benchmax.envs.base_env.BaseEnv`:
- `SearchEnv` — base class with `chunk_overlap_reward_function()`: rewards ≥25% text overlap between model answer and reference chunks; penalizes ≥4 tool calls with 0 reward
- `CorporaSearchEnv` / `TpufSearchEnv` — concrete implementations that expose a `search_corpus` BM25 tool
- Extend `SearchEnv` to build custom environments with different tools while reusing the base reward logic

**`trainer/`** — Training job orchestration via `train()`:
1. Uploads JSONL dataset to blob storage
2. Zips environment class + local module dependencies and uploads
3. Launches training job on the CGFT platform, returns `experiment_id`

**`traces/`** — Agentic trace import and processing via the `TraceAdapter` Protocol:
- `BraintrustTraceAdapter` — fetches traces from Braintrust, groups spans by `root_span_id`, extracts messages via 3-strategy heuristic
- `LangfuseTraceAdapter` — (planned) Langfuse trace import with Basic auth
- Each adapter produces `NormalizedTrace` objects: provider-agnostic, OpenAI-style `{role, content, tool_calls}` messages

Processing pipeline (all provider-agnostic, operates on `NormalizedTrace`):
1. `detect_system_prompt()` / `detect_tools()` — auto-detect from traces
2. `build_training_examples()` — window messages into prompt/completion pairs (cumulative context)
3. `apply_heuristic_filters()` — drop short/trivial turns (returns `FilterResult` with reasons)
4. `apply_pivot_filter()` — PivotRL-based LLM counterfactual analysis to keep only decision-critical turns
5. `split_dataset()` — shuffled train/eval split with min 16 samples guard

Adding a new provider: create `traces/<provider>/adapter.py` implementing `connect`, `list_projects`, `fetch_traces`, add to `traces/registry.py`.

**`multi_model/`** — Utilities for calling multiple LLM providers and comparing pricing/responses.

### Key Data Types

- `Chunk` / `ChunkCollection` (`chunkers/models.py`)
- `QADataPoint` / `QADataset` / `ReferenceChunk` (`qa_generation/models.py`)
- `ChunkSource` Protocol (`corpus/source.py`) — implemented by both corpus backends
- `SearchEnv` (`envs/search_env.py`) — base RL environment; extend this for custom environments
- `TraceAdapter` Protocol (`traces/adapter.py`) — implemented by trace provider backends
- `NormalizedTrace` / `TraceMessage` / `ToolCall` (`traces/adapter.py`) — canonical trace data model
- `TrainingExample` / `PivotRating` (`traces/processing.py`, `traces/pivot.py`) — pipeline output types

### External Integrations

- **CGFT Platform** — training job management and blob storage
- **Corpora API** — document storage and BM25 search
- **Turbopuffer** — vector database
- **Braintrust** — agentic trace import (`api.braintrust.dev`)
- **OpenAI API** — LLM for QA generation (primary)
- **benchmax** — RL training framework (`BaseEnv`, `ToolDefinition`, `StandardizedExample`)

## Design Principles

This library is used across diverse agentic trace types (customer service, code agents, research agents, non-English). Code must be generalizable, not tuned to one domain.

### No hardcoded language or cultural assumptions
- No English stop word lists, locale-specific date formats, or currency symbols baked into library code. Use universal patterns (e.g., `\d+` for all numbers) instead of domain-specific regex.
- If a heuristic only works for English natural-language text, it doesn't belong in the library. Expose a threshold parameter and let the algorithm handle the rest.

### Design the composition layer, not just the leaves
- When building a set of related functions (filters, adapters, processing steps), design how they compose before implementing the individual pieces. A pipeline runner, combined result type, or chaining mechanism should exist from day one.
- Without a composition layer, every consumer (wizard, notebook, Modal worker) writes its own ad-hoc accumulator, and they diverge.

### Fail loudly, never silently reorder or fix
- If a user provides an invalid configuration (e.g., wrong ordering of pipeline steps), raise an error explaining the constraint. Don't silently reorder or "correct" it — the caller won't learn the constraint and the behavior becomes invisible.

### Structured metadata over string parsing
- Return types that carry structured metadata (e.g., `DropReason(filter, reason, detail)`) instead of encoding information into strings that consumers have to parse with `startswith()` hacks.

### Shared utilities need their own tests
- Extracting shared code (e.g., HTTP retry, tokenization) is only valuable if the shared utility has dedicated tests covering its edge cases. Tests that exercise it transitively through callers don't count — they only test the happy path.

### Preserve caller expectations
- Functions in a pipeline should preserve input ordering unless there's a documented reason not to. If a function must reorder (e.g., for determinism), document it explicitly.

### External API code must be tested against real APIs
- Unit tests with mocked HTTP responses cannot validate query formats, column names, auth flows, pagination behavior, or rate limit handling. Mocks return whatever you tell them to — they don't catch invalid SQL, wrong endpoint paths, or missing fields.
- This applies to all external integrations: trace adapters (Braintrust, Langfuse), corpus backends (Turbopuffer, Pinecone, Chroma), the CGFT platform API (training jobs, blob storage), and any future provider.
- Any PR that changes fetch logic, query construction, column lists, pagination, or retry behavior must pass integration tests before merge. This is non-negotiable.
- Reviewers: if a PR touches provider API interaction and the diff contains no integration test changes, that is a red flag. Do not approve without verifying the new behavior works against the real API.
- Integration tests use `@pytest.mark.integration` and are skipped in CI. Run locally: `pytest -m integration` (credentials loaded from `.env.test`). See `.env.test.example` for required keys per provider.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cgftinc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
