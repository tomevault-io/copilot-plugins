## memoriki

> This is a personal LLM-powered knowledge base following Andrej Karpathy's "LLM Wiki" pattern.

# LLM Wiki - Schema

## Overview
This is a personal LLM-powered knowledge base following Andrej Karpathy's "LLM Wiki" pattern.
The LLM maintains a structured, interlinked markdown wiki compiled from raw source documents.

## Directory Structure
```
llm-wiki/
  idea-file.md        # Karpathy's original idea file (reference)
  CLAUDE.md            # This schema file (instructions for the LLM)
  raw/                 # Immutable source documents (LLM reads, NEVER modifies)
    assets/            # Downloaded images
  wiki/                # LLM-generated wiki (LLM owns this entirely)
    index.md           # Content catalog - updated on every ingest
    log.md             # Chronological activity log (append-only)
    entities/          # Pages about specific people, companies, products
    concepts/          # Pages about ideas, patterns, frameworks
    sources/           # Summary pages for each ingested source
    synthesis/         # Cross-cutting analysis, comparisons, insights
```

## Rules
1. **NEVER modify files in `raw/`** - they are immutable source of truth
2. **ALWAYS update `wiki/index.md`** after creating or updating any wiki page
3. **ALWAYS append to `wiki/log.md`** after any operation (ingest, query, lint)
4. **Use [[wiki-links]]** to connect related pages
5. **Add YAML frontmatter** to every wiki page:
   ```yaml
   ---
   title: Page Title
   type: entity|concept|source|synthesis
   sources: [list of raw/ files referenced]
   related: [[linked-pages]]
   created: YYYY-MM-DD
   updated: YYYY-MM-DD
   ---
   ```
6. Log format: `## [YYYY-MM-DD] operation | Description`

## Operations

### Ingest
When user drops a new source into `raw/`:
1. Read the source document
2. Discuss key takeaways with user
3. Create a summary page in `wiki/sources/`
4. Create or update entity pages in `wiki/entities/`
5. Create or update concept pages in `wiki/concepts/`
6. Update `wiki/index.md`
7. Append entry to `wiki/log.md`

### Query
When user asks a question:
1. Read `wiki/index.md` to find relevant pages
2. Read relevant wiki pages
3. Synthesize answer with citations to wiki pages
4. If answer is valuable, offer to save as `wiki/synthesis/` page

### Lint
When user requests health check:
1. Find contradictions between pages
2. Identify orphan pages (no inbound links)
3. Find concepts mentioned but lacking own page
4. Check for stale information
5. Suggest new sources to investigate
6. Log the lint pass

## Language
Write wiki content in the same language as the source material.
Communicate with user in their preferred language.

---
> Source: [AyanbekDos/memoriki](https://github.com/AyanbekDos/memoriki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
