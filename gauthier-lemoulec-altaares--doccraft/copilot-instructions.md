## doccraft

> Python documentation platform for writing structured, dynamic technical docs for non-technical stakeholders (PMs, sales).

# doccraft

Python documentation platform for writing structured, dynamic technical docs for non-technical stakeholders (PMs, sales).

## Structure

- `content/` — YAML data store (glossary, scenarios, solutions) and Markdown pages
- `doccraft/` — Python package (loader, compute, renderer, API)
- `docs/` — MkDocs source; hand-written pages + generated output under `docs/generated/`
- `mkdocs.yml` — MkDocs site config

## Common commands

```bash
# Build the static site
python -m doccraft build

# Live-reload dev server
python -m doccraft serve

# Start the read/write REST API (port 8000)
python -m doccraft api
```

## API

Base URL: `http://localhost:8000`

| Method | Path | Description |
|--------|------|-------------|
| GET | `/glossary` | All glossary entries |
| GET | `/glossary/{acronym}` | Single entry |
| GET | `/scenarios/{doc_id}` | Scenarios with EH |
| GET | `/coverage/{doc_id}` | Full coverage table |
| GET | `/pages/{path}` | Raw Markdown page |
| PUT | `/glossary/{acronym}` | Upsert glossary entry |
| POST | `/scenarios/{doc_id}` | Replace all scenarios |
| PUT | `/scenarios/{doc_id}/{id}` | Upsert single scenario |
| POST | `/coverage/{doc_id}` | Replace coverage data |
| PUT | `/pages/{path}` | Write/update Markdown page |

## Content conventions

- Scenarios: `content/scenarios/{doc_id}.yaml`
- Solutions + coverage: `content/solutions/{doc_id}.yaml`
- Glossary: `content/glossary/hardware.yaml`
- Pages: `content/pages/{doc_id}/{page}.md`

## Computation

- **EH** (Expected Harm) = `risk × probability`
- **Coverage total** = EH-weighted average across all scenarios for each solution column

## Memory policy

Follow the parent project's memory policy (`projects_factory/CLAUDE.md`): never write memory files unless the user explicitly asks to remember something.

---
> Source: [gauthier-lemoulec-altaares/doccraft](https://github.com/gauthier-lemoulec-altaares/doccraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
