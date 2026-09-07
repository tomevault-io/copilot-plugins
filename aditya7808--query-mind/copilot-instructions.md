## query-mind

> QueryMind is a Retrieval-Augmented Generation (RAG) system for answering natural-language questions grounded in company policy documents. It combines hybrid retrieval (TF-IDF + BM25 + FAISS), an abstention guardrail, and both extractive and LLM-based answer generation.

# CLAUDE.md — QueryMind RAG System

## Project Overview

QueryMind is a Retrieval-Augmented Generation (RAG) system for answering natural-language questions grounded in company policy documents. It combines hybrid retrieval (TF-IDF + BM25 + FAISS), an abstention guardrail, and both extractive and LLM-based answer generation.

**Primary user interface:** Streamlit (`app.py`)
**Environment:** Python 3.10+, Windows, local development

---

## Architecture

```
app.py (Streamlit UI)
    └── src/
        ├── ingestion/   (loaders → chunker → metadata)
        ├── retrieval/  (sparse + dense → fusion → HybridRetriever)
        ├── guardrail/  (AbstentionGuardrail)
        ├── generation/ (extractive OR llm)
        └── evaluation/ (metrics → logger)
```

### Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| **Hybrid retrieval** — TF-IDF + BM25 + dense embeddings fused with min-max normalization | TF-IDF/BM25 handle keyword matches; embeddings handle semantic similarity. Neither alone is sufficient for policy QA. |
| **Min-max normalization before fusion** | TF-IDF cosine similarity, BM25 raw scores, and FAISS inner-product live on very different scales. Min-max [0,1] makes weighting interpretable and consistent. |
| **Weighted fusion (0.25 / 0.25 / 0.5) by default** | Semantic embeddings capture meaning better for policy language; keywords catch exact rule references. 50% weight to embeddings, 50% split evenly between sparse methods. |
| **Abstention guardrail at top-score threshold** | Policy errors (wrong leave days, wrong notice period) are worse than no answer. Guardrail prevents hallucination by declining low-confidence queries. |
| **Extractive QA as default** | No API key needed; predictable, cite-able answers; fast. LLM mode is optional for more natural responses. |
| **RAGAS-style metrics** | Context Precision/Recall, Faithfulness, Answer Relevance cover the full retrieval → generation pipeline without requiring human judges per query. |

---

## Critical Paths

### Adding a new document format
Edit `src/ingestion/loaders.py`:
1. Add loader function (e.g., `_load_pptx`)
2. Add extension to `SUPPORTED_FORMATS` tuple
3. Add dispatch in `load_document()`

### Changing retrieval weights
- **Runtime:** Use Streamlit sidebar sliders (changes `config.retrieval.hybrid_weights`)
- **Permanent:** Edit `config.py` → `RetrievalConfig.hybrid_weights`
- **Formula:** `fused = α·TF-IDF_norm + β·BM25_norm + γ·embedding_norm` where α+β+γ=1.0

### Switching LLM backend
- Set `config.generation.mode` to `"openai"`, `"anthropic"`, or `"extractive"`
- In Streamlit sidebar: select from the "Generation Mode" dropdown
- API keys: set in `.env` file or environment variables

### Calibrating guardrail threshold
- Default: 0.42 (generates), 0.30 (declines)
- Increase threshold → fewer hallucinations, more "insufficient information"
- Decrease threshold → more answers, risk of hallucination
- Run `data/logs/guardrail_log.jsonl` analysis to find the right value

---

## Data Files

| Path | Purpose | Auto-generated |
|------|---------|---------------|
| `data/policies/` | Input: put PDFs/DOCX/MDs here | No |
| `data/chunks/chunks.jsonl` | Chunk metadata | Yes (on re-index) |
| `data/indices/faiss_index.bin` | FAISS vector index | Yes (on re-index) |
| `data/indices/tfidf_matrix.npz` | TF-IDF matrix | Yes (on re-index) |
| `data/indices/bm25.pkl` | BM25 parameters | Yes (on re-index) |
| `data/logs/eval_log.jsonl` | Per-query metrics | Yes (auto) |
| `data/logs/guardrail_log.jsonl` | Guardrail decisions | Yes (auto) |

---

## Module Interfaces (Do Not Break)

### `src/retrieval/dense.py` — HybridRetriever.search()
- **Input:** `query: str`, `top_k: int`
- **Output:** `List[Dict]` with keys: `chunk_id`, `score_fused`, `score_tfidf`, `score_bm25`, `score_embedding`
- **Important:** Returns `List[Dict]`, NOT `List[FusedResult]`. Code consuming this should use dict access (e.g., `r["score_fused"]`).

### `src/retrieval/fusion.py` — standalone functions
- `linear_fusion()`, `reciprocal_rank_fusion()`, `hybrid_search()` take score dicts, not retriever objects.
- These are helper functions for the `HybridRetriever` class, not the primary interface.

### `src/ingestion/chunker.py` — Chunk dataclass
- Fields: `id`, `text`, `source_file`, `page_number`, `section_heading`, `byte_start`, `char_start`, `chunk_index`, `total_chunks`
- All fields are strings/ints/None — no nested objects.

### `src/guardrail/abstain.py` — GuardrailResult
- `decision.value` is `"answer"`, `"warn"`, or `"decline"`
- Use `format_guardrail_response(result)` for user-facing text.

### `src/generation/llm.py` — GeneratedAnswer
- Fields: `answer_text`, `citations`, `model_used`, `tokens_used`
- `citations` is a list of chunk dicts.

---

## Common Tasks

### Re-index after adding documents
Click "Re-index Documents" in the Streamlit sidebar, or call:
```python
from src.ingestion.loaders import batch_load_documents
from src.ingestion.chunker import create_chunks_from_documents
from src.retrieval.dense import HybridRetriever
documents = batch_load_documents("data/policies/")
chunks = create_chunks_from_documents(documents)
retriever = HybridRetriever()
retriever.fit([c.text for c in chunks], [c.id for c in chunks])
```

### Run evaluation on existing log
```python
from src.evaluation.metrics import EvaluationLogger
logger = EvaluationLogger()
stats = logger.get_statistics()
print(stats["avg_scores"])
```

### Add a custom metric
1. Add method to `src/evaluation/metrics.py` → `EvaluationMetrics` class
2. Call it in `from_components()` or add to `RAGEvaluator.evaluate_run()`
3. Update evaluation log schema if needed

---

## Dependencies (why they matter)

| Package | Used In | Why |
|---------|---------|-----|
| `scikit-learn` | `sparse.py` | TF-IDF vectorizer |
| `rank-bm25` | `sparse.py` | BM25 Okapi (auto-installed fallback: manual impl) |
| `sentence-transformers` | `dense.py` | all-MiniLM-L6-v2 embeddings |
| `faiss-cpu` | `dense.py` | Fast ANN vector search (numpy fallback if missing) |
| `torch` + `transformers` | `extractive.py` | RoBERTa-base-squad2 for span extraction |
| `pymupdf` | `loaders.py` | PDF text extraction |
| `mammoth` | `loaders.py` | DOCX → text |
| `beautifulsoup4` | `loaders.py` | HTML cleaning |
| `openai` | `llm.py` | GPT generation |
| `anthropic` | `llm.py` | Claude generation |
| `streamlit` | `app.py` | UI framework |

---

## Performance Notes

- **First run:** Sentence-Transformer model downloads on first use (~90MB)
- **Embedding time:** ~50ms per chunk on CPU; ~2ms on GPU
- **Re-indexing:** 100 chunks ≈ 5s on CPU (embedding is the bottleneck)
- **FAISS index type:** `"flat"` is exact search (default). Use `"ivf"` for >10K chunks.
- **Evaluation:** Heuristic metrics are fast (<10ms/query). LLM-as-judge metrics require API calls.

---

## Gotchas

- **FAISS not installed:** `dense.py` falls back to pure numpy cosine similarity (slower for large corpora)
- **rank-bm25 not installed:** `sparse.py` uses a manual BM25 implementation (slower)
- **No API key:** System falls back to extractive mode automatically
- **Empty policies directory:** Streamlit shows a warning; retrieval always returns empty
- **Evaluation needs ground truth:** Current metrics use heuristic overlap. For production, annotate a test set with gold chunks.
- **Guardrail threshold too high:** All queries return "insufficient information" — lower threshold in sidebar

---
> Source: [Aditya7808/Query_Mind-](https://github.com/Aditya7808/Query_Mind-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
