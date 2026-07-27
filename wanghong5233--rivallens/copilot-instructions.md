## agent-runtime-contracts

> Production runtime contracts for LLM, RAG, tool, and async Agent flows


# Agent Runtime Contracts

## Durability

- Persist user-visible intent before starting expensive or failure-prone generation, parsing, retrieval, indexing, or tool execution.
- Any async or long-running Agent path needs explicit states such as `accepted`, `running`, `failed`, `timed_out`, and `completed`.
- Failed requests must preserve enough state for retry, audit, and user trust: provider, model/tool, error code, retryability, and attempt id.

## External dependency semantics

- Do not convert provider auth, quota, parser billing, index, network, schema, or MCP failures into empty success values.
- Classify provider-level failures separately from model-level failures; do not serially exhaust the same failed provider.
- Every fallback must record why it happened and whether output quality changed.

## Retrieval and tool grounding

- RAG, research, and tool-using answers must be traceable to retrieved chunks, source metadata, web evidence, or tool traces.
- Frontend loading text must match the actual backend stage; never show retrieval, parsing, or generation states that did not occur.
- LLM output consumed by code must use structured schemas and fail loudly on invalid shape.

## User-facing failure contract

- Prefer visible failed/retryable states over generic empty screens, `Network Error`, or silent disappearance.
- A retry should create a new attempt unless the backend guarantees idempotent resume semantics.

---
> Source: [wanghong5233/RivalLens](https://github.com/wanghong5233/RivalLens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
