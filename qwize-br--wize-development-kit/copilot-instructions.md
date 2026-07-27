## wize-shard-doc

> core skill: Shard Doc


# Shard Doc

# Shard Doc

Splits large markdown (PRD, architecture, etc.) into addressable shards under `{doc-name}.shards/`.

## Why
Agents quote shards by id (`prd.shards/AC-5.md`) instead of pulling the whole document.

## Behavior
- Splits at headings (`##` by default; configurable).
- Each shard gets `id` from heading slug + frontmatter.
- Original doc becomes the index.

---
> Source: [qwize-br/wize-development-kit](https://github.com/qwize-br/wize-development-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
