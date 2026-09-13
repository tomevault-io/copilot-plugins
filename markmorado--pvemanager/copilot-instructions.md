## pvemanager

> Git commit messages (`git commit -m`) must always be written in English, regardless of the language used elsewhere in the conversation.

## Commit message language

Git commit messages (`git commit -m`) must always be written in English, regardless of the language used elsewhere in the conversation.

## No co-author trailer

Do not add a `Co-Authored-By:` trailer (or any co-authorship line) to commit messages in this project.

## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

Rules:
- For codebase questions, first run `graphify query "<question>"` when graphify-out/graph.json exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).

---
> Source: [markmorado/pvemanager](https://github.com/markmorado/pvemanager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
