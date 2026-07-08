## aria

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project purpose

ARIA is a local RAG pipeline over the **PLU bioclimatique de Paris** PDF corpus. The primary use case is answering urban planning questions (zoning rules, setbacks, heights, mixed-use programmes) grounded in official regulatory documents.

## Common commands

```bash
# Activate the venv first
source .venv/bin/activate

# Install / reinstall in editable mode
pip install -e ".[dev]"

# Ingest PDFs (incremental by default — reuses unchanged files)
aria-rag ingest
aria-rag ingest --max-files 5        # quick test
aria-rag ingest --rebuild            # force full rebuild

# Query — retrieval only (no API cost)
aria-rag ask "question" --no-llm --top-k 8 --family reglement_ecrit

# Query — with LLM synthesis
aria-rag ask "question" --backend openai   # needs OPENAI_API_KEY in .env
aria-rag ask "question" --backend ollama   # needs Ollama running locally

# Patch doc_family without re-embedding (after indexer changes)
python scripts/patch_doc_family.py

# Run tests
pytest
```

## Architecture

The pipeline has two phases: **ingest** and **ask**.

**Ingest** (`indexer.py`):
1. `loader.py` — walks `Ressources/` recursively, extracts text from PDFs with `pypdf`
2. Text is chunked (default 1200 chars / 200 overlap) and filtered by `min_alpha_ratio` to drop scanned/garbage pages
3. `infer_doc_family()` maps folder path fragments to one of 6 families (`reglement_ecrit`, `reglement_graphique`, `rapport_presentation`, `oap`, `padd`, `annexes`)
4. Embeddings are built with `sentence-transformers/paraphrase-multilingual-mpnet-base-v2` and stored in a FAISS `IndexFlatIP` (cosine similarity on normalized vectors)
5. A BM25Okapi index is also built and pickled alongside
6. A manifest tracks file size + mtime for incremental re-ingestion

**Ask** (`retriever.py`):
1. Query is encoded with the same embedding model
2. Both FAISS (semantic) and BM25 (lexical) are searched independently with `fetch_k = limit × 10`
3. Results are fused with **Reciprocal Rank Fusion** (RRF_K=60)
4. Optional `--family` filter restricts the candidate set before both searches
5. Top-k hits are passed to `llm.py` as context for answer synthesis

**LLM** (`llm.py`): supports OpenAI (`gpt-4.1-mini` default) and Ollama (`gemma3:4b` default). The system prompt instructs the model to answer only from provided context.

## Corpus structure

```
Ressources/PLU bioclimatique/
  Règlement/Pièces écrites/     → reglement_ecrit   (~5055 chunks)
  Règlement/Documents graphiques/ → reglement_graphique (~667 chunks)
  Rapport de présentation/      → rapport_presentation (~3616 chunks)
  OAP/                          → oap (~290 chunks)
  PADD/                         → padd (~148 chunks)
  Annexes/                      → annexes (~2205 chunks)
```

Files not matching any of the above folder names get `doc_family = "other"` (~5046 chunks, mostly root-level PDFs like the Code de l'urbanisme).

## Known issues / gotchas

- **macOS NFD encoding**: folder names with accented characters (é, è, î…) are stored as NFD by HFS+. The `infer_doc_family()` function normalizes paths to NFC before matching — this must be preserved whenever `DOC_FAMILIES` keys are edited.
- **Stale index**: if `Chunk` dataclass fields change, existing `chunks.json` will fail to deserialize. Run `aria-rag ingest --rebuild` or use `scripts/patch_doc_family.py` for lightweight fixes that don't require re-embedding.
- **FAISS k=0 guard**: `retriever.py` returns `[]` early if the family filter matches no chunks, avoiding a FAISS assertion error.
- **HF_TOKEN warning**: the sentence-transformers model loads from cache; the unauthenticated HF Hub warning is harmless.

---
> Source: [titesriz/aria](https://github.com/titesriz/aria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
