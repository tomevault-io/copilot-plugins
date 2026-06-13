## python-conventions

> Goal: the editor and type checker should "remember" names, signatures, and


# Python Conventions

Goal: the editor and type checker should "remember" names, signatures, and
data shapes so humans don't have to. Make intent explicit in types, not comments.

## Type annotations are mandatory

Every function gets full annotations: all parameters AND the return type.
A reader (or AI) must understand inputs/outputs from the signature alone.

```python
# BAD - what goes in? what comes out?
async def aretrieve(self, query, top_k):
    ...

# GOOD
async def aretrieve(self, query: str, top_k: int) -> list[Chunk]:
    ...
```

## Never pass structured data as an untyped dict

If a dict has known keys, give it a `TypedDict` (use `total=False` when keys
accumulate across pipeline stages). This makes keys autocomplete and typos fail
type-checking. See `agent/metadata_schema.py` and `rag/base.py` for the pattern.

```python
# BAD
def build(meta: dict): ...        # which keys? nobody knows

# GOOD
class ChunkMeta(TypedDict, total=False):
    chunk_id: str
    chunk_index: int
def build(meta: ChunkMeta): ...
```

## Docstrings explain WHY, not WHAT

Public functions get a docstring with Args/Returns and any non-obvious intent
or trade-off. Do NOT add comments that just narrate the code (`# loop over x`).

---
> Source: [ccJoeyClaire/composer-agentic_rag](https://github.com/ccJoeyClaire/composer-agentic_rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
