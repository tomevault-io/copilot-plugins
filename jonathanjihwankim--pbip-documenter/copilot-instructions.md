## pbip-documenter

> Handles: table, column, measure, hierarchy, partition, relationship, role, expression

# PBIP Documenter

## Project Overview
Browser-based documentation generator for Power BI PBIP/TMDL semantic models. Part of the `pbip-*` tool family by Jihwan Kim (Microsoft MVP).

## Architecture
- **Vanilla JS, no build step** — single-page app deployed to GitHub Pages
- **File System Access API** — reads PBIP folders directly in the browser (Chrome/Edge/Opera)
- **No backend** — all processing happens client-side
- **Dark mode** — CSS custom properties with `[data-theme="dark"]` + `prefers-color-scheme` auto-detect
- **Responsive** — sidebar collapses to toggle button at 768px

## File Structure
- `index.html` — SPA shell with Mondrian/De Stijl themed UI
- `styles.css` — App styling with CSS variables (light + dark themes, responsive breakpoints)
- `app.js` — UI logic, File System Access API integration, event handlers, diagram export routing
- `tmdl-parser.js` — Line-by-line state machine parser for TMDL files
- `visual-parser.js` — PBIR visual.json parser (extracts field references)
- `m-parser.js` — M expression parser (data sources, parameters, 15+ connectors)
- `doc-generator.js` — Output formatting (Markdown, HTML, JSON)
- `diagram.js` — SVG rendering (relationship diagrams, visual usage maps) with dynamic star-schema layout
- `lineage-engine.js` — Dependency graph builder (data sources → tables → measures → visuals)
- `lineage-diagram.js` — SVG lineage visualization (full, trace, impact, column impact)
- `drawio-exporter.js` — draw.io XML export (ERD + lineage diagrams)
- `mermaid-exporter.js` — Mermaid syntax export (erDiagram + flowchart)

## Diagram Export System
All diagram views have a unified toolbar with zoom controls and export buttons:
- **SVG download** — standalone SVG with embedded fonts, explicit dimensions from viewBox
- **draw.io export** — mxGraph XML with `shape=table` containers, ER cardinality arrows, star-schema layout
- **Mermaid export** — copies to clipboard; falls back to .mmd file download
- Export routing: `app.js` `_handleDiagramExport()` → `_exportDiagramSVG/Drawio/Mermaid()`
- Container map in `_exportDiagramSVG()` maps diagram types to DOM container IDs

## TMDL Parser
State machine with states: IDLE → TABLE_BODY → PROPERTIES → EXPRESSION
Handles: table, column, measure, hierarchy, partition, relationship, role, expression
Key challenges: multi-line DAX (indentation-based), backtick blocks, quoted names

## Testing
- `tests/test-runner.html` — browser-based test suite (no Node.js required)
- `scripts/generate-sample-large.html` — generates enterprise sample from D:\sample_powerbi (internal testing only, not exposed to users)
- Enterprise test dataset: D:\sample_powerbi (61 tables, 87 relationships, 246 measures, 542 visuals)
- Contoso test dataset: D:\Contoso\Contoso (10 tables, 11 measures, 15 visuals)

## Related Repositories
- `isHiddenInViewMode` — PBIR Visual Manager (Van Gogh theme)
- `pbip-impact-analyzer` — Impact Analysis + Safe Refactoring (Picasso Cubism theme)

## Conventions
- Same sponsor integration pattern as sibling repos (GitHub Sponsors + Buy Me a Coffee)
- Sponsor toast shows once per session after first download
- Footer cross-links to other tools
- Generated documents include "Generated with pbip-documenter" watermark
- Enterprise sample data is for internal testing only — not exposed in the UI

---
> Source: [JonathanJihwanKim/pbip-documenter](https://github.com/JonathanJihwanKim/pbip-documenter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
