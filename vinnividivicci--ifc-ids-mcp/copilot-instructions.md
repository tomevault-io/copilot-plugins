## ifc-ids-mcp

> **Context:** MCP server for creating buildingSMART IDS (Information Delivery Specification) files with 100% IDS 1.0 XSD compliance using IfcTester. Read `constitution.md` for core principles.

# IDS MCP Server - Agent Guidelines

**Context:** MCP server for creating buildingSMART IDS (Information Delivery Specification) files with 100% IDS 1.0 XSD compliance using IfcTester. Read `constitution.md` for core principles.

## Build/Lint/Test Commands

```bash
# Run all tests with coverage (must pass 95%)
pytest tests/ --cov=src/ids_mcp_server --cov-report=html --cov-fail-under=95

# Run single test file
pytest tests/unit/test_config.py -v

# Run single test function
pytest tests/unit/session/test_manager.py::test_get_or_create_session_new -v

# Format code (100 char line length)
black src/ tests/

# Lint code
ruff check src/ tests/
```

## Code Style Guidelines

**Imports:** stdlib → third-party → local. Use `from typing import` for types. Always import `Context` from `fastmcp`.

**Formatting:** Black (100 char lines). Type hints required on all functions. Use `Optional[T]` for nullable params.

**Async:** All MCP tools are `async def`. Always accept `ctx: Context` parameter (auto-injected by FastMCP).

**Naming:** snake_case functions/variables, PascalCase classes, UPPER_CASE constants. Tool names match function names.

**Error Handling:** Raise `ToolError` from `fastmcp.exceptions` for tool errors. Log with `await ctx.info/error/warning()`.

**TDD Required:** RED-GREEN-REFACTOR. Write failing test first, minimal implementation, then refactor. 95%+ coverage enforced.

**IDS Operations:** ALWAYS use IfcTester library (`ids.Ids()`, `ids.Specification()`, etc.). NEVER generate XML manually.

**Session Management:** Use `ctx.session_id` (never manual session params). Store in `get_session_storage()`.

**Validation:** Three levels: (1) Tool input → IfcTester object, (2) Object structure, (3) XML export with `ids.from_string(xml, validate=True)`.

**Documentation:** Docstrings on all public functions. Include Args, Returns, Raises sections. Reference constitution.md for principles.

## Critical IDS 1.0 Schema Constraints

**⚠️ One Entity Per Applicability:** IDS 1.0 XSD allows only ONE entity facet per specification's applicability section. Create separate specifications for each IFC element type. Early validation in `validators.py` catches this.

**⚠️ Property Set Required:** The `property_set` parameter is REQUIRED when adding property facets (use `Pset_Common` for custom properties, or IFC standard sets like `Pset_WallCommon`). Early validation enforces this.

## Essential Rules

**✅ DO:** Use IfcTester for ALL IDS operations • Write tests BEFORE implementation (TDD) • Use `ctx: Context` in all tools • Validate with `ids.from_string(xml, validate=True)` • Maintain 95%+ coverage • Follow type hints • Use async/await

**❌ DON'T:** Generate XML manually (use IfcTester) • Use manual `session_id` parameters (use Context) • Skip tests or write tests after implementation • Accept <95% coverage • Create non-deterministic outputs • Bypass IfcTester with custom XML parsing

---
> Source: [vinnividivicci/ifc-ids-mcp](https://github.com/vinnividivicci/ifc-ids-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
