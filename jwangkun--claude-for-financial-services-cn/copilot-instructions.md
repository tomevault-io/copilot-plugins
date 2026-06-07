## claude-for-financial-services-cn

> China-market equivalent of the main `financial-services` repo. All data sourced from open-source Chinese datasets (AkShare / Tushare).

# CLAUDE.md — China Financial Services Plugins

China-market equivalent of the main `financial-services` repo. All data sourced from open-source Chinese datasets (AkShare / Tushare).

## Source of truth

- **Skills** are authored in `vertical-plugins/china-finance/skills/`.
- **Agent system prompts** are at `agent-plugins/<slug>/agents/<slug>.md`.
- Agent plugins bundle vendored copies of their skills. **Never edit bundled copies directly.**

## Core data layer

Two MCP servers run locally:

```bash
# Start the AkShare data server (A-share market data)
python3 mcp-servers/akshare-mcp/server.py

# Start the China news server
python3 mcp-servers/china-news-mcp/server.py
```

Both default to stdio transport. For deployment, use `--transport sse --port 8000`.

AkShare documentation: https://akshare.akfamily.xyz/

## Commands

```bash
# Validate everything before push
python3 scripts/check-china.py

# Test cookbook structure (dry-run deployment)
bash scripts/test-china-cookbooks.sh

# Deploy a cookbook (dry-run or live)
bash scripts/deploy-managed-agent.sh <slug> [--dry-run]

# After editing a skill in vertical-plugins/, propagate to all agents that bundle it
python3 scripts/sync-china-skills.py

# Cross-agent handoff orchestration (reference implementation)
python3 scripts/orchestrate.py

# Validate worker output against a JSON schema
python3 scripts/validate.py <output.json> <schema.json|schema.yaml>
```

## Key conventions

- Agent frontmatter: every `agents/*.md` must have YAML `---` with `name` + `description`.
- Tool references in agent frontmatter use `mcp__akshare__*` and `mcp__china-news__*` syntax.
- Stock codes follow A-share conventions: 6-digit codes, no exchange prefix (e.g., "600519", "000001").

## Agents (4 China-market)

- `china-pitch-agent` — A-share pitch book (comps, DCF, LBO, deck)
- `china-market-researcher` — A-share sector research
- `china-earnings-reviewer` — A-share earnings analysis
- `china-model-builder` — DCF, LBO, 3-statement for Chinese equities

## Dependencies

Install per-server:
```bash
pip install -r mcp-servers/akshare-mcp/requirements.txt
pip install -r mcp-servers/china-news-mcp/requirements.txt
```

---
> Source: [jwangkun/claude-for-financial-services-cn](https://github.com/jwangkun/claude-for-financial-services-cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-07 -->
