## 02-chunking-and-splitting-rules

> Chunking and Text Splitting - Use chunking that preserves coherence while fitting model/token budgets. `src/ai/text-splitter.ts`


# Chunking and Text Splitting Rules

Last reviewed: 2025-08-11

Use chunking that preserves coherence while fitting model/token budgets.

## Defaults
- **Recursive splitter** with ordered separators (e.g., `\n\n`, `\n`, ` `. then fallback) and overlap to reduce boundary loss.
- **Semantic splitter** only when embeddings are available and budgeted; otherwise stick to recursive.
- **Chunk size/overlap**: Tune per task; start with conservative sizes, validate with recall/precision in retrieval.

## Project Guidance
- Use `createResearchSplitter()` in `src/ai/text-splitter.ts`.
- Avoid external calls in tests; mock `generateTextEmbedding()` when exercising semantic splitting.
- Ensure each chunk carries sufficient context for summarization prompts; consider chunk expansion around headings.

## Anti-Patterns
- Overlapping too much (wastes tokens) or too little (context loss at boundaries).
- Large chunks that exceed downstream prompt budgets when batched.

## References
- Chunking strategies (Pinecone): (https://www.pinecone.io/learn/chunking-strategies/)

---
> Source: [ssdeanx/deep-research-mcp-server](https://github.com/ssdeanx/deep-research-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
