## django-admin-mcp

> For comprehensive AI agent guidelines, coding standards, architecture details, and development workflows, see [AGENTS.md](../AGENTS.md) in the repository root.

# Copilot Instructions

For comprehensive AI agent guidelines, coding standards, architecture details, and development workflows, see [AGENTS.md](../AGENTS.md) in the repository root.

## Quick Reference

- **Project**: django-admin-mcp - Exposes Django admin models to MCP clients via HTTP
- **Python**: 3.10+ with type hints
- **Linting**: `ruff check .` and `ruff format .`
- **Type checking**: `mypy django_admin_mcp/`
- **Tests**: `pytest --cov=django_admin_mcp`
- **Line length**: 120 characters

## Key Conventions

- ModelAdmin inheritance order: `MCPAdminMixin, admin.ModelAdmin` (mixin first)
- All handlers go in `django_admin_mcp/handlers/`
- Protocol types defined in `django_admin_mcp/protocol/types.py`
- Always respect Django admin permissions for CRUD operations

---
> Source: [7tg/django-admin-mcp](https://github.com/7tg/django-admin-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
