## mcp-odoo

> Guidance for Claude Code (claude.ai/code) working in this repository.

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository.

This is the **upstream source** for `odoo-mcp` — the Odoo MCP server published
to PyPI as `odoo-mcp` (`uvx odoo-mcp`) and consumed by downstream projects as
their `odoo` MCP server.

**This repo is the canonical place to fix odoo-mcp.** Any bug or enhancement
discovered while using odoo-mcp from a consumer project should be fixed here,
not worked around downstream. Consumers run the **published** package — a fix
only reaches them once it is republished, or once their MCP config is
repointed at a checkout (`uvx --from /path/to/mcp-odoo odoo-mcp`). Use that to
verify a fix end-to-end.

Start with [CONTRIBUTING.md](CONTRIBUTING.md) and
[docs/adding-a-tool.md](docs/adding-a-tool.md) for the dev workflow, layering
contracts, and quality gates (`pytest`, `ruff`, `mypy`, `lint-imports`, Docker
smoke). Never bypass the gated write workflow or the field-ACL choke points.

---
> Source: [tuanle96/mcp-odoo](https://github.com/tuanle96/mcp-odoo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
