## agenticocr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Does

A multimodal RAG (Retrieval-Augmented Generation) evaluation pipeline for document understanding benchmarks. It retrieves relevant page regions from documents using visual reranking and agentic OCR extraction, generates answers via LLM, and evaluates results. Currently supports two benchmarks: **MMLongBench-Doc** (mmlong) and **FinRAGBench-V** (finrag).

## Commands

### Running the Pipeline

The pipeline has three sequential stages. Each uses `--config` for YAML configuration and shares common args defined in `bootstrap.py`:

```bash
# 1. Retrieval: rerank pages and extract evidence elements
python run_retrieval.py --config configs/template.yaml --num_threads 10

# 2. Generation: produce LLM answers from retrieval output
python run_generation.py --config configs/template.yaml --num_threads 10

# 3. Evaluation: score retrieval and generation quality
python run_evaluation.py --config configs/template.yaml --num_threads 4 --evaluation_task all
```

### Batch Experiments

`batch_run.sh` runs all three stages across prompt/setting combinations. It copies `configs/test_qwen.yaml`, modifies context flags via sed, and runs sequentially.

### Aggregate Results

```bash
python scripts/aggregate_results.py --output_base_dir outputs/my_experiment --prompts prompt0 --settings_start 1 --settings_end 7
```

### Dependencies

```bash
pip install -r requirements.txt   # Python 3.12, CUDA 12.8
```

## Architecture

### Three-Stage Pipeline

`bootstrap.py` is the central orchestrator — it defines shared CLI args, parses YAML configs (via omegaconf), and wires up all components through `initialize_components()`. The three `run_*.py` scripts are thin wrappers that call into `bootstrap.py` and the agent/loader layer.

### Data Model (`src/loaders/base_loader.py`)

- **`PageElement`** — atomic retrieval unit with bbox (0-1000 normalized coords), content, corpus_id/path, and optional crop_path
- **`StandardSample`** — normalized input sample (qid, query, data_source, gold_answer, gold_pages, gold_elements, extra_info)
- **`BaseDataLoader`** — abstract base requiring `load_data()`, `pipeline()`, `evaluate_retrieval()`, `evaluate_generation()`

### Benchmark Loaders (`src/loaders/`)

- **`MMLongLoader`** — loads MMLongBench-Doc samples from `samples.json`, converts PDFs to page images, uses LLM-based answer extraction with `MMLONG_EXTRACT_PROMPT_TEMPLATE` and custom `eval_score()` (ANLS, float equality, exact match)
- **`FinRAGLoader`** — loads FinRAG queries/qrels by language (ch/en/both), supports FAISS embedding index and bbox evaluation mode
- **`loader_common.py`** — shared extraction helpers: `run_extractor_with_optional_judger()`, `extract_page_elements_from_data()`, element-level IoU metrics

Both loaders share the same patterns: `pipeline()` → rerank pages → extract elements via agentic OCR → return `PageElement` list. They also both implement expansion recall (adding adjacent pages to top-k results).

### Agent Layer (`src/agents/`)

- **`RAGAgent`** — core orchestrator with `retrieve(sample)` and `generate(query, elements)` methods. Uses OpenAI-compatible API. `build_context_message()` constructs multimodal prompts from PageElements with configurable context flags (`use_page`, `use_crop`, `use_ocr`, `use_ocr_raw`, `use_ocr_both`, `use_page_ocr`)
- **`AgenticOCR`** — multi-turn tool-use agent loop. Sends page images to an LLM that calls `image_zoom_and_ocr_tool` via `<tool_call>` XML tags. Parses tool calls, executes crops/OCR, feeds results back
- **`MinerUTool`** — `ImageZoomOCRTool` handles image crop/rotate/OCR via MinerU VL server. `MinerUBboxExtractor` does direct bbox-to-text extraction. Both use MinerUClient for OCR inference

### Retrieval (`src/recalls/`)

- `Qwen3VLEmbedder` — visual embedding model for FAISS indexing
- `Qwen3VLReranker` — visual reranker (local model or HTTP client mode via `reranker_api_base`)

### Utilities

- `src/utils/common.py` — cache read/write (atomic), JSONL I/O, `run_parallel()` (ThreadPoolExecutor + tqdm), `smart_resize()`
- `src/utils/llm.py` — `LLMCaller` wrapper around OpenAI client with retry
- `src/utils/eval.py` — standalone bbox evaluation script with visualization (IoU_Min, IoU_EM, Page Accuracy)

### Configuration

All args can be set via CLI or YAML config (`--config`). YAML values become parser defaults, CLI flags override them. Config template at `configs/template.yaml`. Sensitive keys (api_key, etc.) are redacted when saved to output_dir.

### Caching

Both retrieval and generation stages cache per-sample results as individual JSON files in `cache_retrieval_results/` and `cache_generation_results/` under output_dir. Generation skips cached entries that have valid (non-error) answers.

## Sample Filtering and Selection

All sample filtering is centralized in `initialize_components()` (`bootstrap.py`), which serves as the single authority. Filters execute in a fixed order:

```
loader.load_data()          ← full dataset
       │
       ▼
  ① --filter               ← qid whitelist (bad_case JSON file)
       │
       ▼
  ② --category_filter      ← category filter (comma-separated, case-insensitive)
       │
       ▼
  ③ --limit                ← take first N from filtered result
       │
       ▼
  loader.samples            ← final working set (single source of truth)
```

**`--filter`** accepts a JSON file with target qids. Supported formats: plain qid list `["0","1"]`, object list `[{"qid":"0",...}]` (e.g. bad_case output), or nested dict `{"qids":[...]}` / `{"items":[...]}`.

**`--category_filter`** matches against `extra_info["evidence_sources"]` (MMLong, string repr of list parsed via `ast.literal_eval`) and `extra_info["category"]` (FinRAG, string or list). Normalization: `str(name).split("-")[0].split(" ")[0].lower()`, so `"Table-complex"` matches `"table"`. Example: `--category_filter "Table,Chart"`.

**`--limit`** executes last, so `--limit 10 --category_filter Table` means "first 10 of the Table category", not "Table items among the first 10".

### How each stage consumes filtered samples

- **Retrieval** (`run_retrieval.py`): directly uses `loader.samples`.
- **Generation** (`run_generation.py`): loads data from `retrieval_results.json` (which lacks category metadata), then calls `filter_items_by_samples(data_items, loader.samples)` to project all filtering via qid intersection.
- **Evaluation** (`run_evaluation.py`): `_match_results_to_samples()` builds a map from `loader.samples` (already filtered) and only matches qids present in that map.

Adding a new filter only requires changing `initialize_components()` — all three stages inherit it automatically.

## Key Context Flags

The `use_*` flags in RAGAgent control what evidence is included in the generation prompt:
- `use_page` — full page image
- `use_page_ocr` — full-page MinerU OCR text
- `use_crop` — cropped region images from extracted elements
- `use_ocr` — OCR text content from elements
- `use_ocr_raw` — raw MinerU OCR instead of model-extracted content
- `use_ocr_both` — include both raw OCR and model-extracted content

---
> Source: [opendatalab/AgenticOCR](https://github.com/opendatalab/AgenticOCR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
