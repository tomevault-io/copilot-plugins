## vinted-scraper

> **This is the highest-priority instruction in this file.**

## MANDATORY: Graphify First Rule

**This is the highest-priority instruction in this file.**

Whenever the user asks a question about the codebase, architecture, modules, dependencies, or how something works — **YOU MUST run `graphify query "<the user's question>"` FIRST** before reading any source files, grepping, or using ripgrep.

**NO EXCEPTIONS except:**
1. The user explicitly says "do not use graphify"
2. The task is about fixing stale/incorrect graph output
3. `graphify-out/graph.json` does not exist

**Why:** Querying the graph costs ~40× fewer tokens than grepping raw files. If you skip graphify and start grepping, the user pays for every file you open. The graph gives you scoped, relevant files immediately.

**After graphify query:** Read only the files the graph points you to. Do not browse broadly.

**After code changes:** Run `graphify update .` to keep the graph current (AST-only, no API cost).

**Specific tools:**
- Broad questions: `graphify query "<question>"`
- Relationships between two things: `graphify path "<A>" "<B>"`
- Focused concept: `graphify explain "<concept>"`
- Only if graphify returns nothing useful: fall back to `rg` on the specific files it surfaced

## graphify (reference)

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

When the user types `/graphify`, use Graphify before doing anything else. If a dedicated `skill` tool is available, invoke it with `skill: "graphify"`; otherwise run the `graphify` CLI directly.

- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- Dirty graphify-out/ files are expected after hooks or incremental updates; dirty graph files are not a reason to skip graphify.

## Vinted token-saving rules

- Use `docs/AI_CONTEXT.md` as the compact project map for future sessions.
- Do not scan `data/`, `wiki/`, or `graphify-out/` unless the user explicitly asks.
- Prefer `rg` or Graphify queries over opening broad documentation sets.
- Keep generated graph files local; this branch tracks the Graphify setup, not the generated graph payload.

---
> Source: [alegosto01/Vinted_Scraper](https://github.com/alegosto01/Vinted_Scraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
