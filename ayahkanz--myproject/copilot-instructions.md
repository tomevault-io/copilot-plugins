## myproject

> Mode: B (GitHub/Repo) + C (Business/Project)

# FTI Multi-App Ecosystem — LLM Wiki

Mode: B (GitHub/Repo) + C (Business/Project)
Purpose: Knowledge base for 16+ FTI UII web apps — architecture, decisions, app status, Pranata TI portfolio
Owner: 4h3
Created: 2026-07-06

## Wiki Knowledge Base
Path: /Users/4h3/myproject

When you need context not already in this project:
1. Read wiki/hot.md first (recent context cache, ~500 words)
2. If not enough, read wiki/index.md (full catalog)
3. If you need domain details, read wiki/<domain>/_index.md
4. Only then drill into specific wiki pages

Do NOT read the wiki for general coding questions.

## Structure

```
myproject/
├── .raw/                   # immutable source documents
│   ├── articles/
│   ├── transcripts/
│   ├── screenshots/
│   ├── data/
│   └── assets/
├── wiki/                   # LLM-generated knowledge base
│   ├── index.md            # master catalog
│   ├── log.md              # append-only operation log
│   ├── hot.md              # recent context cache (~500 words)
│   ├── overview.md         # executive summary
│   ├── sources/
│   ├── entities/
│   ├── concepts/
│   ├── domains/
│   ├── comparisons/
│   ├── questions/
│   └── meta/
├── _templates/
├── _attachments/
├── WIKI.md                 # schema & instructions
└── CLAUDE.md               # this file
```

## Conventions
- All notes: YAML frontmatter (type, status, created, updated, tags)
- Wikilinks: `[[Note Name]]` format
- `.raw/` is read-only — never modify sources
- `wiki/index.md` — update on every ingest
- `wiki/log.md` — append-only, new entries at TOP

## Operations
- Ingest: drop source in .raw/, say `ingest [filename]`
- Query: ask any question — Claude reads index first
- Save answer: `/save`
- Lint: `lint the wiki`
- Full scaffold: `/wiki`

---
> Source: [ayahkanz/myproject](https://github.com/ayahkanz/myproject) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
