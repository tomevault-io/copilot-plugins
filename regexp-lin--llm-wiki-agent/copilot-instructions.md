## wiki-conventions

> LLM Wiki — base conventions for wiki structure and naming


# Wiki Base Conventions

## Schema
Read `WIKI_SCHEMA.md` at the project root for the full wiki data model.

## Immutability
- `raw/` is immutable — never modify source documents
- `wiki/` is agent-owned — only the agent writes here
- `graph/` is auto-generated output

## Naming
- Source pages: `wiki/sources/<kebab-case>.md`
- Entity pages: `wiki/entities/<TitleCase>.md`
- Concept pages: `wiki/concepts/<TitleCase>.md`
- Use `[[PageName]]` wikilinks to cross-reference

## Index & Log
- Update `wiki/index.md` on every content change
- Append to `wiki/log.md` with format: `## [YYYY-MM-DD] <op> | <title>`

## Available Workflows
Use the corresponding skill by saying one of these triggers:
- **Ingest**: `ingest <file>` or `ingest` (batch) — ingest source documents
- **Query**: `query: <question>` — search the knowledge base
- **Lint**: `lint` — health-check the wiki
- **Graph**: `build graph` — build the knowledge graph

---
> Source: [regexp-lin/llm-wiki-agent](https://github.com/regexp-lin/llm-wiki-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
