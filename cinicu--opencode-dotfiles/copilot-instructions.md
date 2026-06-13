## opencode-dotfiles

> Use Perplexity MCP tools for search, research, and reasoning tasks.

# AGENTS.md

## PERPLEXITY-MCP-START

Use Perplexity MCP tools for search, research, and reasoning tasks.

Available tools:
- `perplexity_search` - Quick web search with citations
- `perplexity_ask` - Synthesized answers (mode: concise/copilot)
- `perplexity_reason` - Multi-step reasoning
- `perplexity_research` - Deep research reports
- `perplexity_compute` - Computer/ASI mode for complex computations

Sources: `web` (default), `scholar`, `social`

Examples:
- "Search scholar sources for recent RAG evaluation papers" → `perplexity_search` with `sources: ["scholar"]`
- "What are developers saying about Cursor vs Windsurf?" → `perplexity_ask` with `sources: ["social"]`
- "Deep research on code review automation" → `perplexity_research` with `sources: ["scholar", "web"]`

## PERPLEXITY-MCP-END

---
> Source: [cinicu/opencode-dotfiles](https://github.com/cinicu/opencode-dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
