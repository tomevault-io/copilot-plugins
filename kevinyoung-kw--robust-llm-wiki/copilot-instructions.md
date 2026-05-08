## robust-llm-wiki

> This repository is about Karpathy-style LLM-Wiki as an operating pattern, not just as a slogan.

# CLAUDE.md

This repository is about Karpathy-style LLM-Wiki as an operating pattern, not just as a slogan.

If you only read one file, start here and then continue to:

1. `schema/robust-llm-wiki-CLAUDE.md`
2. `schema/SPEC.md`

## The Original Karpathy Baseline, Made Explicit

Do not reduce the original pattern to a vague sentence like "preserve the kernel."

The concrete baseline is:

1. There are three layers: `raw sources`, `wiki`, and `schema`.
2. `raw sources` are immutable. Read them, cite them, synthesize from them, but do not rewrite them.
3. `wiki` is the maintained markdown knowledge base. The agent is expected to write and maintain this layer.
4. `schema` is the contract that teaches the agent how to operate the wiki safely.

The original workflow is also concrete:

1. `ingest` reads a new source,
2. writes or updates a source summary page,
3. updates related entity, concept, comparison, overview, or synthesis pages,
4. updates `index.md`,
5. appends an operation entry to `log.md`.

The original `query` behavior is:

1. search the wiki first,
2. often read `index.md` first,
3. drill into relevant pages,
4. synthesize an answer from maintained wiki pages,
5. optionally file valuable answer artifacts back into the wiki.

The original `lint` behavior is:

1. check contradictions across pages,
2. look for stale claims superseded by newer sources,
3. find orphan pages or weak cross-references,
4. spot important concepts that should exist as pages but do not yet,
5. propose concrete maintenance actions.

`index.md` and `log.md` are not decoration in this pattern.

1. `index.md` is the content catalog and first navigation surface.
2. `log.md` is the append-only operational timeline.

If an agent ignores all of that and only remembers "wiki + links + ingest/query/lint", it has not actually understood the Karpathy baseline.

## Repository-Specific Policy

After loading this file, follow the full operator policy in `schema/robust-llm-wiki-CLAUDE.md`.

That file defines:

1. `draft -> stable` rules,
2. agent team ownership rules,
3. risk classes,
4. lint rhythm,
5. schema extension rules,
6. model allocation guidance.

---
> Source: [KevinYoung-Kw/robust-llm-wiki](https://github.com/KevinYoung-Kw/robust-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
