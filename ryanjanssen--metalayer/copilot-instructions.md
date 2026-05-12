## metalayer

> A self-learning context layer for LLM analytics. Your data model is the `context/` folder — `.md` files with `[[wikilinks]]`.

# Metalayer

A self-learning context layer for LLM analytics. Your data model is the `context/` folder — `.md` files with `[[wikilinks]]`.

## First Time Setup

If the vault has no source files yet (check `context/sources/`), the user needs to set up before anything else works. Run `metalayer validate` — if it shows 0 sources, walk the user through the setup skill. This discovers available data connections, creates source files, and optionally runs a first import.

## Sources and Data Access

Sources are vault files (type: source) in `context/`. Each source tells you how to access a warehouse or database — which CLI command to run, or which MCP server to use. **Use these tools to run queries.** You have access to them in your environment.

Every view references a source with `source: [[source_name]]`. When you need to verify data or execute SQL, resolve the source file and use the tool it describes. If no source files exist, read the setup skill (Step 2) to create one before doing anything else.

**Never write a context file without verifying its claims against real data using the source.** See the writing-context skill — Verify Before Writing.

## When a user asks a data question

Read and follow the ask-data skill. This is the master workflow. It will tell you to:

1. Search the vault for relevant context (`metalayer search`)
2. Navigate by reading context files (`metalayer get-context`)
3. Generate SQL via 5-sub-agent consensus (sql-consensus skill)
4. Execute the SQL via the source's tool
5. Reflect on results (reflection skill)
6. Log the query and propose vault updates if needed

Do not skip steps. Do not write SQL without first gathering context from the vault.

**Every data answer MUST include an answer card** — run `metalayer answer-card` to generate it. See the "Answer Card" section in the ask-data skill for the exact format. This is not optional.

## When a user asks to add or edit context

Read the writing-context skill for format rules. Always run `metalayer audit` as a preflight check before writing. Never write vault files without showing the user and getting approval.

## When a user asks to import from a source

Read the update-from skill for the general workflow, then read the relevant preset from `utils/imports/`.

## When a user asks to audit or validate

Run `metalayer validate` for static checks. Read the context-audit skill for the deeper semantic audit the LLM performs.

## Architecture

- `context/` — the vault: `.md` files with `[[wikilinks]]`, indexed by QMD
- `utils/queries/` — query memory (also indexed by QMD)
- `skills/` — agent skill files (NOT indexed — read on demand)
- `utils/imports/` — import preset instructions (NOT indexed — read on demand)
- `src/metalayer/` — Python tooling (CLI + tools)
- `config.yaml` — project configuration

## Key Rules

- Files are identified by filename (stem). `[[orders.revenue]]` resolves to `orders.revenue.md`
- `type` is the only required frontmatter field
- Seven types: source, view, field, relation, concept, topic, lore
- The vault is the table whitelist: no `view.md` = invisible to the LLM
- Fields use dotted names: `{view}.{field}.md`
- Never silently modify vault files — always show changes and get approval
- Query memory is a 500-file ring buffer; accepted queries are permanent
- `vault_overview.md` in `context/lore/` is auto-generated — never edit manually

## CLI Tools

```
metalayer get-context <name>     # Read a context file with its links
metalayer get-topic <name>       # Resolve a topic to all its files
metalayer search <query>         # Search the vault via QMD
metalayer validate               # Static validation checks
metalayer audit                  # Context audit
metalayer sync                   # Re-index and validate
metalayer update-from <source>   # Import from a source
metalayer answer-card            # Render answer metadata card
metalayer log-query              # Log query + trigger learning
metalayer frequent               # Show most-used vault objects
```

## Development

- Use `uv` for package management (never `pip`)
- Python 3.11+
- Run tests: `uv run pytest src/metalayer/tests/`
- Run validation: `uv run metalayer validate`
- Install in dev mode: `uv pip install -e ".[dev]"`

---
> Source: [ryanjanssen/metalayer](https://github.com/ryanjanssen/metalayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
