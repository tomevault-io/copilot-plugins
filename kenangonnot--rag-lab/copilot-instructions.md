## rag-lab

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project purpose

Personal learning repo for RAG techniques — naive RAG through advanced pipelines (hybrid retrieval, re-ranking, multi-query, HyDE, RAGAS eval). See `README.md` for the experiments roadmap; this is an evolving sandbox, not a product.

## Commands

```bash
# Install
pip install -r requirements.txt

# Configure
cp .env.example .env  # fill in OPENAI_API_KEY

# Build the FAISS index — run the notebook first to populate data/processed/faiss_index
jupyter notebook experiments/01_naive_rag.ipynb

# Launch the chat UI (requires an existing index at data/processed/faiss_index)
python -m chat.app            # → http://localhost:8080

# Docker alternative
docker compose up
```

There is no test suite or linter wired up yet.

## Architecture

The pipeline is split into four layers that compose top-down. Each layer exposes a small class and is wired together in `chat/app.py`:

1. **`ingestion/`** — `DocumentLoader` (PDF/TXT/directory → `list[Document]`) → `Chunker` (fixed or recursive) → `Embedder` (OpenAI). Build-time only; output is a persisted FAISS index.
2. **`retrieval/`** — `VectorStore` wraps FAISS (`build`/`save`/`load`/`as_retriever`). `Retriever` is a thin facade over it. The index is saved to `data/processed/faiss_index` and loaded at app startup.
3. **`generation/`** — `RAG_PROMPT` (ChatPromptTemplate with `{context}`/`{question}`) and `RAGChain`, built with **LangChain LCEL**: `{context: retriever | format_docs, question: passthrough} | prompt | ChatOpenAI | StrOutputParser`. Exposes sync `run` and async `arun`.
4. **`chat/`** — `ChatUI` (NiceGUI on port 8080, async submit via `RAGChain.arun`). `chat/app.py` is the entry point: loads `Settings`, builds `Embedder` → `VectorStore.load` → retriever → `RAGChain` → `ChatUI`.

**`config/settings.py`** is a Pydantic `BaseSettings` that centralises all knobs (`openai_api_key`, `embedding_model`, `llm_model`, `chunk_size`, `chunk_overlap`, `top_k`) loaded from `.env`. Always read config through `get_settings()` — do not read env vars directly elsewhere.

**Index lifecycle**: notebooks/scripts in `experiments/` do the ingestion and **write** the index; `chat/app.py` **reads** it. If `data/processed/faiss_index` is missing, the app will not start — run `experiments/01_naive_rag.ipynb` first.

## Conventions

- Python 3.11+ syntax — use `str | None`, not `Optional[str]`.
- Short docstrings on classes and public methods; no module-level side effects.
- KISS — prefer the simplest working implementation over abstractions for hypothetical future experiments. New RAG techniques should land as new modules (e.g. `retrieval/advanced/…`) rather than generalising existing ones prematurely.
- The `data/raw/` and `data/processed/` directories are gitignored except for `.gitkeep`.

---
> Source: [kenanGonnot/rag-lab](https://github.com/kenanGonnot/rag-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
