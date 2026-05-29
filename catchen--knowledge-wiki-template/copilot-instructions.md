## knowledge-wiki-template

> This workspace is a personal knowledge base — a collection of Markdown notes and documents.

# Knowledge Base

This workspace is a personal knowledge base — a collection of Markdown notes and documents.

## Search

The `qmd` MCP server is connected and indexes this entire folder as the `knowledge` collection. Use it for any search or lookup task:

- `qmd__query` — search using a `searches` array of typed sub-queries. Three types: `lex` (BM25 keyword; supports `"quoted phrase"` and `-negation`), `vec` (semantic/meaning-based), `hyde` (hypothetical answer passage, 50-100 words). First sub-query gets 2× weight — combine types for best recall. Key params: `intent` (disambiguates query), `minScore`, `limit` (default 10).
- `qmd__get` — fetch a single document by path, docid (`#abc123`), or path with line offset (`file.md:100`). Supports `fromLine`, `maxLines`, `lineNumbers`. Search results include a `line` field — use `fromLine: line-20, maxLines: 80` to read surrounding context.
- `qmd__multi_get` — fetch multiple documents by glob pattern or comma-separated list. Supports `maxBytes` (skip large files; default 10 KB), `maxLines`, `lineNumbers`.
- `qmd__status` — check index health and document count.

**Default behavior:** When asked about something that might be in the notes, search `qmd` before answering from training knowledge. If `qmd` is unavailable or returns insufficient results, fall back to `rg` (ripgrep) to search the files directly.

## Folder Conventions

| Folder  | Contents                                                                                   |
| ------- | ------------------------------------------------------------------------------------------ |
| `Wiki/` | AI-generated wiki — see below. Do not edit manually.                                       |

Source folders (e.g. `Notes/`, `Ideas/`, `Docs/`) are defined by the repo owner. Treat any folder not listed above as user-maintained source content.

## Wiki

The `Wiki/` directory implements the [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) pattern: rather than re-synthesizing knowledge on every query, AI incrementally builds a persistent, interconnected set of markdown pages from the source documents. All content in `Wiki/` is generated and maintained by the skills below — do not edit it manually.

| Skill                       | When to run                                                                                                                |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `/knowledge-wiki-summary`   | After adding or editing source files — generates or refreshes summary files in `Wiki/Summaries/`                           |
| `/knowledge-wiki-concept`   | After running summary — creates or updates concept articles in `Wiki/Concepts/`                                            |
| `/knowledge-wiki-synthesis` | Periodically after accumulating new concepts — discovers cross-cutting connections and writes synthesis articles           |
| `/knowledge-wiki-lint`      | Periodically, especially after reorganizing source files — repairs orphan summaries, broken wikilinks, and orphan concepts |
| `/knowledge-wiki-merge`     | Periodically — interactive session to identify and merge duplicate concept articles                                        |
| `/knowledge-wiki-enrich`    | Periodically — expands thin concept articles using web search                                                              |

---
> Source: [CatChen/knowledge-wiki-template](https://github.com/CatChen/knowledge-wiki-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
