## em-rag

> Read `AGENTS.md` first. It is the canonical agent workflow for this repository.

# Gemini Guide

Read `AGENTS.md` first. It is the canonical agent workflow for this repository.

Key rules:

- Use `list_docs` to discover indexed documents when the RAG state is unknown.
- Use `search_docs` before answering questions about chips, registers, datasheets, SDK APIs, source files, or board behavior.
- Do not index by default. Only call `index_doc` when the user explicitly asks to add, rebuild, refresh, or index documents.
- Do not remove indexed documents unless the user explicitly asks for removal.
- Prefer the default `pymupdf` PDF backend for normal PDFs; use MinerU for scanned or complex-layout PDFs.

---
> Source: [LeoKemp223/em_rag](https://github.com/LeoKemp223/em_rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
