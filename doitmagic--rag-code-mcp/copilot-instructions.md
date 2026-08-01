## rag-code-mcp

> **For any information about the code (location, structure, logic, or usage), you MUST use RagCode MCP tools. Never guess code details from memory; always search the local index first.**

# RagCode MCP - Semantic Search Rules
# This file is automatically managed by RagCode MCP.

## ⚖️ The Golden Rule
**For any information about the code (location, structure, logic, or usage), you MUST use RagCode MCP tools. Never guess code details from memory; always search the local index first.**

## Available Tools
- search_code: Primary entry point for semantic search.
- get_function_details: Get full implementation of a function.
- find_type_definition: Get struct/interface definitions.
- list_package_exports: See what a module offers.
- search_docs: Find project documentation.

## Usage Guidelines
- Always provide 'file_path' to tools to ensure they detect the correct project context.
- Use 'hybrid_search' if looking for exact variable names or error messages.
- If the tool says "workspace not indexed", use 'index_workspace' once.

---
> Source: [doITmagic/rag-code-mcp](https://github.com/doITmagic/rag-code-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
