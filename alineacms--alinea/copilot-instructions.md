## alinea

> This project uses Alinea, a Git-based headless CMS. The schema and workspaces

## Alinea

This project uses Alinea, a Git-based headless CMS. The schema and workspaces
are configured in `{cmsFile}`, content is stored as JSON files in `content/`.

- Docs: `node_modules/alinea/docs/` holds the documentation that matches the
  installed version, start at `index.md`. Prefer it over what you remember.
- Content changes: use the tools of the `alinea` MCP server, configured in
  `.mcp.json`. It works while `alinea dev` runs, validates content and keeps
  the dashboard in sync. If its tools are not available, ask the user to
  enable the server before editing content files by hand.
- Never edit generated files: `public/admin.html`, `public/admin/` and
  `@alinea/generated`.

---
> Source: [alineacms/alinea](https://github.com/alineacms/alinea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
