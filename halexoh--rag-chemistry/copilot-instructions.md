## rag-chemistry

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Retrieval-Augmented Generation system over a chemistry-of-coatings corpus (436 sources: full textbooks, vendor data sheets, papers, standards — bilingual ES/EN), built **from scratch with no RAG framework** (no LangChain/LlamaIndex) so every stage — chunking, embedding, indexing, retrieval, generation — is explicit and inspectable. Runs 100% locally (Ollama + FAISS + sentence-transformers) on Apple Silicon. Every design decision, including dead ends, is documented in `docs/00` through `docs/07` (in Spanish) — read the relevant one before changing that stage's behavior; this file only summarizes.

## Commands

```bash
# Setup (macOS)
brew install poppler tesseract ollama
ollama pull qwen2.5:7b-instruct
brew services start ollama
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# Full pipeline, in order (each stage reads the previous stage's output)
python -m src.extraction.run        # data/raw/        -> data/processed/*/pages.jsonl
python -m src.chunking.run          # pages.jsonl       -> chunks.jsonl
python -m src.embeddings.run        # chunks.jsonl      -> embeddings.npy
python -m src.indexing.build_index  # embeddings.npy    -> vectorstore/index.faiss + metadata.jsonl
python -m src.evaluation.run_eval   # runs the 15-question test set, writes eval/results/<timestamp>.json

# Tests
pytest                              # whole suite
pytest tests/test_chunker.py        # one module
pytest tests/test_chunker.py::test_name -v   # one test

# Interactive
streamlit run app.py                          # chat UI
jupyter notebook notebooks/pipeline_completo.ipynb   # narrated walkthrough with real outputs
```

There is no lint/format command configured in this repo.

## Pipeline architecture

Each stage is a `src/<stage>/` package with a `run.py` (or `build_index.py`) CLI entrypoint and reads/writes flat files under `data/processed/<book-slug>/` — there is no database. Stages must be re-run in order after any upstream change (e.g. re-chunking requires re-embedding, since `embeddings.npy` row *i* corresponds positionally to `chunks.jsonl` line *i*, with no other key tying them together).

```
data/raw/ (PDFs, not committed)
  -> extraction    (src/extraction) -> data/processed/<slug>/pages.jsonl
  -> chunking      (src/chunking)   -> chunks.jsonl   (token-based, chapter-bounded)
  -> embeddings    (src/embeddings) -> embeddings.npy (row i <-> chunks.jsonl line i)
  -> indexing      (src/indexing)   -> vectorstore/index.faiss + metadata.jsonl (not committed)
  -> retrieval     (src/retrieval)  -> FAISS top-20 -> cross-encoder rerank -> top-5
  -> generation    (src/generation) -> grounded, cited answer via Ollama
  -> evaluation    (src/evaluation) -> deterministic metrics, no LLM-as-judge
```

`src/generation/answer.py::answer_question()` is the single entrypoint that ties retrieval + generation together — the notebook, `app.py`, and `run_eval.py` all call this same function, not the individual stages.

### Extraction (`src/extraction/`)

- Two source layouts, distinguished by **directory name**, not content sniffing: a folder named `[Author_Year]_Title` is a whole book (each PDF inside = one chapter); anything else is a category, and every loose PDF inside becomes its own standalone cited document. This distinction can't be inferred from file contents — both layouts look like "a flat folder of PDFs" on disk.
- Chapter numbers come from filename conventions accumulated empirically from real files (see the doc comment at the top of `chapters.py` for the exact patterns) — when a new book doesn't chapter-detect correctly, add a pattern there rather than special-casing the book.
- `pdf_text.py` auto-falls-back to Tesseract OCR per-page when PyMuPDF's native text extraction returns near-nothing (scanned pages) — this is a per-page decision, not a per-book one.
- Single-PDF books derive chapters from the embedded PDF table of contents (`chapters.py::chapter_map_from_toc`); multi-PDF books derive them from filenames.

### Chunking (`src/chunking/`)

- Token-based (`tiktoken`, `cl100k_base`), 600 tokens with 100 overlap, and **never crosses a chapter boundary** — chunking is done independently per `(book, chapter)` group. If you change chunk size/overlap, re-run embeddings and indexing after.

### Embeddings & retrieval (`src/embeddings/`, `src/retrieval/`, `src/indexing/`)

- Embedding model `intfloat/multilingual-e5-base` requires the `"query: "` / `"passage: "` prefix convention (see `embedder.py`) — omitting it degrades retrieval quality; it's not just a formatting nicety.
- Retrieval is two-stage: FAISS `IndexFlatIP` (bi-encoder, fetches top-20 broadly) -> `BAAI/bge-reranker-base` cross-encoder (precise, rescoring only those 20 down to top-5). Don't skip the reranker or shrink `fetch_k` too close to `top_k` — the entire point is giving the reranker a wide net.
- Two independent exclusion filters run at index-build time only (chunks stay in `chunks.jsonl`/`embeddings.npy` untouched, so filters are reversible): `structural_filter.py` (exact match on known non-content chapter labels: Index, TOC, Front Matter, References...) and `reference_filter.py` (regex heuristic detecting bibliography-shaped text: multiple `(YYYY)` + numbered-line patterns). Both exist because raw similarity search ranks reference lists and back-of-book indexes deceptively high on shared vocabulary alone.
- **Embeddings and reranking are forced onto CPU** (`device="cpu"` explicit in both `embedder.py` and `reranker.py`) — MPS contention with Ollama's own GPU usage caused indefinite stalls under memory pressure on a 16GB machine. Do not "optimize" this back onto MPS without re-reading `docs/04-indexacion.md`.
- **`src/__init__.py` sets `KMP_DUPLICATE_LIB_OK=TRUE` and `OMP_NUM_THREADS=1` before any submodule can import faiss/torch** — faiss-cpu and PyTorch each bundle their own OpenMP runtime, and on macOS the wrong import order between them segfaults the process. Both env vars are required together (one alone reproduces the crash); this is why `src/__init__.py` must run first and why new entrypoints should live under `src/` rather than import faiss/torch before `src` is imported.

### Generation (`src/generation/`)

- `prompt.py` builds a system prompt that forces every factual claim to end in a `[Book, Chapter, p. X-Y]` citation copied verbatim from the chunk header, and instructs the model to refuse rather than guess when excerpts are insufficient. The citation regex in `evaluation/metrics.py` depends on this exact bracket format — changing the format requires updating both places.
- `generate.py` calls local Ollama (`qwen2.5:7b-instruct`) with no framework in between.

### Evaluation (`src/evaluation/`)

- Deliberately **not** LLM-as-judge — metrics are pure string/keyword/regex matching (`metrics.py`) against a small hand-verified 15-case test set (`testset.py`), so evaluating correctness never depends on another model's own correctness.
- `testset.py`'s `expected_book` field is optional and only set where a topic still maps unambiguously to one source; it was written when the corpus had 2 books and many topics now legitimately span multiple valid sources after the archive integration — don't reintroduce `expected_book` for a topic without checking it's still exclusive to one book.
- Out-of-domain metrics are inherently noisy run-to-run (LLM samples at temperature > 0) — don't treat a single eval run's refusal/fabrication rate as ground truth; compare against the range in `docs/07-evaluacion.md`.

## Adding a new source document

Drop it into `data/raw/` following the naming convention described above (`[Author_Year]_Title/` for a book, loose PDFs elsewhere for standalone documents), then run the full pipeline in order from extraction through indexing. See `docs/01-extraccion.md` for edge cases (duplicate files, versioned re-exports, un-chaptered articles in edited volumes).

---
> Source: [Halexoh/RAG-chemistry](https://github.com/Halexoh/RAG-chemistry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
