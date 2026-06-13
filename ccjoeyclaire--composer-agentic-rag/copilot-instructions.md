## metadata-schema

> Metadata dict conventions for the RAG/agent pipeline


# Metadata Conventions

`metadata` dicts flow through many pipeline stages and gain fields as they go.
They are NOT free-form. Two rules keep them from becoming a memory black hole.

## 1. Every metadata field lives in a TypedDict schema

Document all possible keys in a stage-grouped `TypedDict(total=False)`, the same
way `agent/metadata_schema.py` (`AgentMetadata`) does. When you add a field to a
metadata dict, add it to the schema in the SAME change.

```python
class ChunkMeta(TypedDict, total=False):
    # Indexing
    chunk_id: str
    anchor_window: AnchorWindow
    # Retrieve (small-to-big)
    parent_id: str
    matched_chunk_ids: list[str]
```

## 2. Access keys via constants, never string literals

Keys are defined once as module constants (see `parent_builder.py`:
`CHUNK_ID_KEY`, `ANCHOR_WINDOW_KEY`, ...). Always use them.

```python
# BAD - a typo here fails silently at runtime
cid = meta.get("chunk_id")

# GOOD - one source of truth, rename-safe
cid = meta.get(CHUNK_ID_KEY)
```

When introducing a new metadata key: add the constant, add it to the relevant
TypedDict, and update the schema docstring.

---
> Source: [ccJoeyClaire/composer-agentic_rag](https://github.com/ccJoeyClaire/composer-agentic_rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
