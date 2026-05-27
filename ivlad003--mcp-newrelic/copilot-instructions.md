## mcp-newrelic

> MCP server for querying New Relic data via NRQL/NerdGraph.

# MCP NewRelic

MCP server for querying New Relic data via NRQL/NerdGraph.

## Environment Variables

- `NEW_RELIC_API_KEY` — Required. New Relic User API key (starts with `NRAK-`).
- `NEW_RELIC_ACCOUNT_ID` — Required. Numeric New Relic account ID.

## Project Structure

- `app.py` — MCP server (single tool: `query_logs` for NRQL queries via NerdGraph)
- Run with: `uv run app.py`

## Development

- Python 3.13+, managed with `uv`
- Dependencies: httpx, mcp[cli], pydantic

## Claude Code Plugin

Also available as a standalone Claude Code plugin (no MCP server needed):
- Repo: https://github.com/Ivlad003/plugins
- Install: `/plugin marketplace add Ivlad003/plugins`

---
> Source: [Ivlad003/mcp_newrelic](https://github.com/Ivlad003/mcp_newrelic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
