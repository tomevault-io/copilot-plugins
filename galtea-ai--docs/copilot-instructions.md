## docs

> > **SCOPE:** These rules apply when working on files under `docs/`.

# Mintlify Documentation Guidelines

> **SCOPE:** These rules apply when working on files under `docs/`.

## Project Structure

```
docs/
├── docs.json              # Mintlify configuration (navigation, theme)
├── concepts/              # Concept documentation
├── sdk/                   # SDK documentation (api/, tutorials/)
├── code/                  # Code snippets (Python scripts)
├── snippets/              # Reusable MDX snippets
├── images/                # Documentation images
└── logo/                  # Logo assets
```

## Critical Rule: Code Embeds

**NEVER write inline Python code blocks in `.mdx` files.** All code examples live in standalone Python scripts under `docs/code/` so they can be tested in CI/CD.

### How it works

1. Code files live in `docs/code/python/` as full, runnable Python scripts with section markers:
   ```python
   # @start section_name
   result = galtea.products.create(name="My Product")
   # @end section_name
   ```
2. MDX files embed sections:
   ```mdx
   {/* @embed path="code/python/my_file.py" lang="python" section="section_name" */}
   ```

### Code file rules

- Section names: descriptive `snake_case` (e.g., `usage_and_cost_info`)
- File naming: `sdk_api_<service>_<method>.py`, `sdk_tutorials_<topic>.py`, `concepts_<topic>.py`
- Before creating a new code file, check if an existing one covers the topic — add a section to it instead.

## Adding a New Page

1. Create the MDX file with frontmatter (`title`, `description`, `icon`)
2. Add the page path to `docs.json` navigation

## Component and Template Reference

Component usage (Callouts, Tabs, Cards), SDK API reference page template, redirects, and image conventions are documented in `docs/component_reference.md`.

## Architecture Decision Records

Docs-specific ADRs live in `docs/adr/`. Consult them before making structural changes to the documentation.

| ADR | Topic |
|-----|-------|
| `0001-docs-file-renames-and-redirects.md` | How to rename pages: file rename + `docs.json` navigation update + redirect + internal link updates (excluding old changelogs) |

## Local Development

```bash
cd docs && npm install && npm run dev  # http://localhost:3000
```

Validate: `python scripts/run.py` | Build: `python scripts/run.py --build`

---
> Source: [Galtea-AI/docs](https://github.com/Galtea-AI/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
