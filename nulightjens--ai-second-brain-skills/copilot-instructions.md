## ai-second-brain-skills

> This folder is a Karpathy-style LLM wiki. If you are Claude Code, read `CLAUDE.md` instead — it has the full schema. This file is the minimal mirror for OpenAI Codex and other agents.

# LLM Wiki — AGENTS.md

This folder is a Karpathy-style LLM wiki. If you are Claude Code, read `CLAUDE.md` instead — it has the full schema. This file is the minimal mirror for OpenAI Codex and other agents.

## Layout

- `raw/` — source documents. **Read only. Never modify.**
- `wiki/` — your workspace. Create and update markdown files here.
- `wiki/index.md` — catalog of every page. Read this first on any query.
- `wiki/log.md` — append-only operation log.

## Page format

Frontmatter: `title`, `tags`, `sources`, `related`, `last_updated`. Use `[[wikilinks]]` for cross-references. One concept per page.

## Log format

Every operation appends one entry:

```
## [YYYY-MM-DD] <op> | <title>
```

Ops: `ingest`, `query`, `lint`, `update`, `init`.

## Operations

**Ingest**: read source → report takeaways → identify affected pages → update existing + create new → update index → append log.

**Query**: read index → find pages → synthesize with citations → offer to file good answers back as new pages.

**Lint**: scan for contradictions, stale claims, orphans, missing pages, missing cross-refs, data gaps → propose severity-ranked fixes.

## Guardrails

- Never modify files in `raw/`.
- Never delete wiki pages without explicit confirmation — flag instead.
- Never add factual claims without sources.
- Never fabricate citations.
- Never silently resolve contradictions — note them.

---
> Source: [NulightJens/ai-second-brain-skills](https://github.com/NulightJens/ai-second-brain-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
