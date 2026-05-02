## open-agreements

> This extension is for local OpenAgreements workflows using two MCP servers:

# OpenAgreements — Local MCP Extension Context

This extension is for local OpenAgreements workflows using two MCP servers:

1. `contracts-workspace-mcp` (`@open-agreements/contracts-workspace-mcp`)
- Workspace setup planning (`workspace_init`)
- Catalog validation/fetch (`catalog_validate`, `catalog_fetch`)
- Status generation and lint (`status_generate`, `status_lint`)

2. `contract-templates-mcp` (`@open-agreements/contract-templates-mcp`)
- Template discovery (`list_templates`)
- Template inspection (`get_template`)
- Template drafting/fill output (`fill_template`)

## Trust Boundary

Both MCP servers run locally via stdio and operate on local filesystem paths.
No hosted MCP endpoint is required for extension operation.

## Recommended Flow

1. Use `workspace_init` to plan folder structure and collaboration files.
2. Use `list_templates` / `get_template` to pick and inspect the contract template.
3. Use `fill_template` to render a local DOCX output.
4. Use workspace status tools to track draft/executed lifecycle state.

---
> Source: [open-agreements/open-agreements](https://github.com/open-agreements/open-agreements) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
