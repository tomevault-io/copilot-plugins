## crawl4ai-mcp-server

> **ALWAYS remember**: When ANY documentation file is modified (CLAUDE.md, README.md, .cursorrules, etc.), you MUST:

# Project rules for Cursor in `crawler_agent`

## Documentation Memory Rule (CRITICAL)
**ALWAYS remember**: When ANY documentation file is modified (CLAUDE.md, README.md, .cursorrules, etc.), you MUST:
1. **Read the current version** of all related documentation files first
2. **Synchronize changes** across Claude Code (CLAUDE.md) and Cursor (.cursorrules) configurations  
3. **Update project knowledge** to reflect any new patterns, tools, or workflows
4. **Verify consistency** between documentation and actual implementation
5. **Document the change** in git commit messages as a documentation update

## Mission
- Build and expose Crawl4AI functionality as MCP tools consumable by OpenAI Agents SDK, Cursor, and Cloud Code.
- Deliver small, safe, reversible changes with high reliability and clear observability.
- Maintain consistency with Claude Code environment via synchronized documentation.

## Operating mode
- Be a senior engineer: propose a short plan, then execute.
- Prefer minimal diffs; change only what’s needed; avoid unrelated reformatting.
- Preserve existing indentation and style; match the surrounding code.
- Default to strict typing, meaningful names, small functions, early returns.

## Workflow
- Before coding: confirm scope in a numbered plan; call out risky edits for explicit approval.
- Write/expand tests when feasible; keep coverage meaningful (no performative tests).
- After edits: run the smallest checks (build/test/lint) required to validate; capture output.
- Summarize changes and next steps succinctly.

## Tooling rules (Cursor agents)
- Use the repository’s docs for authoritative references:
  - Crawl4AI: `https://docs.crawl4ai.com/`
  - OpenAI Agents SDK MCP: `https://openai.github.io/openai-agents-python/mcp/`
  - Tools reference: `https://openai.github.io/openai-agents-python/tools/`
  - Agent reference (mcp_servers, mcp_config): `https://openai.github.io/openai-agents-python/ref/agent/`
  - MCP intro: `https://modelcontextprotocol.io/introduction`
- When reading local docs:
  - Prefer targeted searches. For `docs/modelcontextprotocol-python-sdk-CLAUDE-MD.txt`, read specific sections only (never the entire file).
  - Use grep-style queries to locate symbols (e.g., `Server(`, `@server.call_tool`, `MCPServerStdio`).
- Parallelize read-only queries to gather context quickly; sequence only when output A is needed for B.

## Design constraints
- Transport: start with stdio MCP server; consider SSE/Streamable HTTP later.
- Contracts: strict, versioned JSON schemas (Pydantic) for tool inputs/outputs.
- Observability: structured logs, tracing spans for `list_tools`/`call_tool`, basic counters and latencies.
- Safety: domain allow/deny lists, robots.txt modes, max depth/TTL/timeouts, proxy/auth guards.
- Performance: reuse browser contexts where possible; cache results (opt-in); avoid deep re-crawls by default.

## Coding conventions
- Python 3.11+; explicit type hints for public APIs; no bare `except`; meaningful error messages.
- Avoid deep nesting; guard clauses first; do not over-catch exceptions.
- Add concise docstrings for public functions/classes; explain intent, not obvious mechanics.
- Do not add comments to explain trivial code; keep code self-documenting.

## Edit discipline
- Do not rename or move files unless requested or essential; if so, include a rollback note.
- Keep unrelated formatting unchanged. Do not mass-reflow lines or reorder imports unless necessary.
- Gate new dependencies behind explicit confirmation; pin versions when adding.

## Security & privacy
- Never enable crawling of internal networks, localhost, or file:// schemes.
- Respect site policies; provide robots.txt compliance modes.
- Redact secrets; never log credentials or PII; sanitize outputs forwarded to LLMs.

## MCP + Crawl4AI specifics (Current Implementation)
- **4 Production Tools**: `scrape`, `crawl`, `crawl_site`, `crawl_sitemap` 
- **Server Location**: `crawler_agent/mcp_server.py` (518 lines, fully implemented)
- **Tool Schemas**:
  - `scrape(url, output_dir?)` → Returns content OR persists with metadata
  - `crawl(seed_url, max_depth?, max_pages?, output_dir?)` → Multi-page crawling  
  - `crawl_site(entry_url, output_dir, ...)` → Full site crawling (always persists)
  - `crawl_sitemap(sitemap_url, output_dir, ...)` → Sitemap-based crawling (always persists)
- **Content Hiding**: Tools with `output_dir` persist to disk and return metadata only (avoids context bloat)
- **Validation**: Use `python -m crawler_agent.smoke_client` for testing

## Validation steps (typical)
- Lint/tests/build minimal run; for MCP stdio server, a smoke run that lists tools and runs `crawl_url` against a simple page.
- Capture outputs and errors; fail fast with actionable messages.

## Docker Integration (Recommended)
- **Docker commands**: Use `./docker-run.sh` helper script for common operations
- **Build and test**: `./docker-run.sh build && ./docker-run.sh test` 
- **Run server**: `./docker-run.sh run` (stdio MCP mode)
- **Development**: `./docker-run.sh dev` (shell access)
- **Benefits**: Zero setup, consistent environment, no dependency conflicts

## Virtual Environment Protocol (Manual Setup Only)
- **ALWAYS activate virtual environment before ANY operation**: `source .venv/bin/activate` or `source venv/bin/activate`
- **Required for**: package installation, running servers, testing, development commands
- **Location**: `<project_root>/.venv` or `<project_root>/venv` (project-specific)
- **Never run Python commands without venv activation** - this causes ModuleNotFoundError

## Remote MCP Server Usage
When using this MCP server from other repositories:
- **Installation**: Clone repository to global location (e.g., `~/.local/mcp-servers/`)
- **Configuration**: Update MCP config with absolute paths to cloned repository
- **Testing**: Use `python -m crawler_agent.smoke_client` to verify setup
- **Documentation**: Copy `.cursorrules` and relevant sections to target repository

## Development Commands (All require venv)
```bash
# Always start with:
source .venv/bin/activate

# Test MCP server:
python -m crawler_agent.smoke_client

# Run server directly:
python -m crawler_agent.mcp_server

# Install dependencies:
pip install -r requirements.txt
```

## Claude Code Integration Notes
- This project supports both Cursor and Claude Code via synchronized documentation
- Claude Code users: See `@CLAUDE.md` for full project context and memory bank
- MCP integration: Native tool calling via `mcp__crawl4ai-mcp__*` functions
- Both environments share `.mcp.json` configuration for project-scoped servers

## Documentation & citations
- When referencing external content, cite links inline using Markdown.
- Update project docs/runbooks when changing public contracts, transports, or observability.
- **Always synchronize changes** between CLAUDE.md and .cursorrules files.

## Commit etiquette
- Clear, scoped commit messages; include rationale and any rollout/rollback notes when changing contracts or infra.
- Mark documentation updates clearly when modifying CLAUDE.md, .cursorrules, or README.md.

---
> Source: [sadiuysal/crawl4ai-mcp-server](https://github.com/sadiuysal/crawl4ai-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
