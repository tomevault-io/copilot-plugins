## miru-code-search

> Miru MCP is the default for conceptual code exploration


# Miru Code Search

DO NOT use Grep, Glob, SemanticSearch, or broad file reads to explore how code works when Miru MCP is available.

Search returns compact snippets (~±15 lines around the best match). When a hit has `truncated: true`, call `expand` with `file_path` and `anchor_line` (or `start_line`) instead of rereading the whole file.

Use Miru MCP tools:
- `search` — one call per question; pass project root as `repo`
- `expand` — more context in the same file when `truncated: true`
- `find_related` — similar code in other files

Allowed native-tool exceptions:
- exact literal lookups such as env vars or quoted strings
- reading a file Miru already identified so it can be edited
- searching outside the indexed repo

---
> Source: [takara-ai/miru-code](https://github.com/takara-ai/miru-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
