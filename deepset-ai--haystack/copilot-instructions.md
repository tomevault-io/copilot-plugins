## haystack

> - Use provider-compatible roles in `haystack/hooks/compaction/` — prefer `user` for synthetic markers

# haystack/hooks/compaction/ Guidelines

## API Design

- Use provider-compatible roles in `haystack/hooks/compaction/` — prefer `user` for synthetic markers
- Name and document compaction settings by their real semantics — retention in `turns`/`steps`, not `messages`; state when compaction runs, what is retained, and which tool-call context is preserved

## General

- Keep `haystack/hooks/compaction/` compactors narrative — move shared indexing, grouping, token counting, and helpers into focused utils

---
> Source: [deepset-ai/haystack](https://github.com/deepset-ai/haystack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
