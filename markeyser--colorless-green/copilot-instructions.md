## colorless-green

> Guidance for agents working on this repo. Keep diagrams consistent and avoid theming issues with MkDocs Material.

# AGENTS.md

## Overview

Guidance for agents working on this repo. Keep diagrams consistent and avoid theming issues with MkDocs Material.

## Filesystem MCP Server Usage

- Use the `filesystem` MCP server to read files within the workspace; keep requests scoped and use relative paths (avoid `node_modules/`, `site/`, large assets).
- Prefer targeted `listDirectory` calls on the specific subfolder you need rather than broad tree walks.
- Avoid fetching binary/large files (images, PDFs); read only the needed sections of large text files to keep responses small.
- Treat the MCP server as read-only; apply edits with normal file operations (e.g., `apply_patch` or your editor).

## Git MCP Server Usage

- Use the `git` MCP server for repo metadata: diffs, history, blame, and status; avoid write operations through MCP.
- Prefer narrow queries: specific file diffs (`git diff -- <file>`), file history (`git log -- <file>`), or short status checks; avoid large history dumps.
- When inspecting changes, target relevant paths (`docs/`, `src/`) instead of the whole tree to reduce output size.
- Do not use the MCP server to stage/commit/push; perform write operations with standard git commands outside MCP.

## Diagram and Asset Policy

- Do **not** use Mermaid flowcharts; render diagrams as images with a white background.
- Compress any image before committing; pre-commit blocks files >500 KB.
- MkDocs Material injects theme variables into Mermaid and forces text to `var(--md-text-color)` in dark mode, which breaks custom colors—avoid Mermaid to prevent this.
- Recommended workflow:
  - Generate the diagram in Mermaid Live Editor (or similar).
  - Export as PNG/SVG with a white background.
  - Compress the image.
  - Place it in an `assets/` directory at the same level or below the Markdown file that references it (e.g., `docs/glossary/three-pillars-of-domain-ai.md` uses `docs/glossary/assets/domain-ai-three-piplines.png`).
  - Embed via Markdown, e.g. `![Pipelines](assets/pipelines.png)`.

## Project Basics

- This is a MkDocs Material site; content lives under `docs/` and navigation is defined in `mkdocs.yml`.
- When adding a new page, update `mkdocs.yml` to include it in the nav.

## Build and Preview

- Local preview: `mkdocs serve`.
- Deployment: push to the remote GitHub repo; Pages build/updates automatically (no `mkdocs gh-deploy` needed).

## Checks and Style

- Run pre-commit locally: `pre-commit run --all-files` (ensures linting, size checks, etc.).
- Tests (if relevant): `poetry run pytest` or repo-specific test commands.
- Markdown: include alt text for images; keep headings/lists markdownlint-friendly.
- Assets: keep images <500 KB and use `assets/` alongside the referencing Markdown; prefer ASCII unless the file already uses Unicode.

## Mermaid MCP Servers (when legacy Mermaid is unavoidable)

- Preferred approach is still image exports (see Diagram and Asset Policy). If you must use Mermaid, follow this tool-driven workflow.
- Choose diagram type via `mermaid-doc-mcp-server.get_diagram_doc("<type>")` (flowchart, sequence, class, state, etc.); do not paste full docs—extract only rules you need.
- Draft Mermaid using those rules; quote labels with parentheses, e.g., `A["Encode (query)"]`.
- Validate every diagram with `mermaid-validator.validateMermaid(diagram)`; fix errors and re-validate until it returns valid.
- Insert only validated code blocks plus a short `_Figure:_` caption explaining what and why.

---
> Source: [markeyser/colorless-green](https://github.com/markeyser/colorless-green) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
