## zvec-mcp

> For repository and implementation questions, first consult the local project knowledge base through the MCP tool search_project_knowledge before doing any broad repository search. If search_project_knowledge returns a tool error or is unavailable, do not silently continue as if the knowledge base had been checked successfully; explicitly mention that the MCP knowledge base was unavailable and then fall back to a targeted grep search. If search_project_knowledge returns no useful results, explain that the knowledge base did not provide a useful hit and then proceed to a focused grep search.

# Agent Instructions

For repository and implementation questions, first consult the local project knowledge base through the MCP tool search_project_knowledge before doing any broad repository search. If search_project_knowledge returns a tool error or is unavailable, do not silently continue as if the knowledge base had been checked successfully; explicitly mention that the MCP knowledge base was unavailable and then fall back to a targeted grep search. If search_project_knowledge returns no useful results, explain that the knowledge base did not provide a useful hit and then proceed to a focused grep search.

> [!NOTE]
> The database indexes source files only. Files with extensions like `.md` (documentation), configuration files (`package.json`, `tsconfig.json`, `vite.config.ts`, `eslint.config.js`), and declaration files (`.d.ts`) are **ignored** by design. For these files or queries about general architecture/configs, skip vector search and use `grep_search` or `view_file` directly.
> If the MCP tool throws locking/concurrency errors (e.g. `Can't lock collection`), it may mean there are stale bridge processes holding the file lock. Proactively list and terminate any stale `node zvec-mcp-bridge.js` processes.
> If the index seems outdated or returns zero results on valid codebase queries, call `initialize_project_knowledge` with `force_rebuild: true` to regenerate the database.

## Required workflow

1. Start with search_project_knowledge.
2. Treat a match as useful only if it directly names the queried symbol, file, or feature and the entry was indexed within the last 30 days; when a match is useful, use it as the primary source of context. If matches are only tangentially related, treat the result as insufficient.
3. If search_project_knowledge returns a tool error, unavailable state, or zero/weak results, explicitly state that the knowledge base was unavailable or did not provide a useful match, then proceed to a focused grep search.
4. If the user explicitly asks for a fresh code inspection, search directly instead of starting with the knowledge base.
5. If both search_project_knowledge and grep return no relevant results, tell the user explicitly that no information was found in the knowledge base or codebase, and ask for clarification such as the specific file path, symbol name, or module.
6. Avoid starting with a broad grep when the knowledge base already contains relevant context.

---
> Source: [ABIvan-Tech/zvec-mcp](https://github.com/ABIvan-Tech/zvec-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
