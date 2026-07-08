## llmwiki

> This repository is a persistent wiki maintained by an LLM agent.

# LLM Wiki Agent Schema

This repository is a persistent wiki maintained by an LLM agent.

## Mission

Turn raw source documents into a structured, interlinked markdown wiki that compounds over time. Prefer updating existing synthesis over creating duplicate notes.

## Repository Rules

### Layers

- `raw/` contains source documents and assets. Treat this directory as immutable. Read from it, but do not modify, rename, or delete files unless the user explicitly asks.
- `wiki/` contains LLM-authored markdown pages. You may create and edit files here freely.
- `AGENTS.md` defines the operating rules for this wiki. Read it before substantial work. Update it only when the user wants to change the workflow.

### Source of truth

- Raw sources are the canonical evidence.
- Wiki pages should distinguish between direct source-backed claims and higher-level synthesis.
- When sources conflict, record the disagreement instead of silently overwriting older claims.

## Wiki Conventions

### Page types

- `wiki/sources/` - one page per source, focused on summary, notable claims, and outbound links
- `wiki/entities/` - durable pages for people, organizations, products, places, works, etc.
- `wiki/concepts/` - durable pages for ideas, themes, methods, debates, and recurring topics
- `wiki/analyses/` - question-driven outputs such as comparisons, timelines, memos, and presentations
- `wiki/meta/` - top-level navigation and maintenance pages

### Linking

- Prefer wiki-style relative markdown links, for example `[Prompt Engineering](../concepts/prompt-engineering.md)`.
- Add links when a page mentions another durable concept or entity that already exists.
- If an important concept or entity appears repeatedly but has no page yet, consider creating one.

### Obsidian conventions

- Keep files plain markdown and Obsidian-friendly.
- When useful, add YAML frontmatter for navigation and Dataview.
- Prefer these frontmatter fields: `title`, `type`, `status`, `created`, `updated`, `tags`, `source_files`.
- Use `type` values such as `source`, `entity`, `concept`, `analysis`, or `meta`.
- Update the `updated` field when making meaningful changes to a page.
- Store downloaded attachments under `raw/assets/`.
- Favor linked pages over inline duplication so graph view stays useful.

### Naming

- Use lowercase kebab-case file names.
- Keep titles human-readable in the first heading.
- Avoid near-duplicate pages; merge into an existing page when scope overlaps heavily.

### Page structure

Use concise sections when helpful. Typical sections include:

- `## Summary`
- `## Key points`
- `## Evidence`
- `## Open questions`
- `## Related pages`
- `## Sources`

For pages that benefit from metadata, use frontmatter like:

```yaml
---
title: Human Readable Title
type: concept
status: active
created: 2026-05-10
updated: 2026-05-10
tags:
  - wiki/concept
source_files:
  - raw/example.md
---
```

Not every page needs every section.

## Required Files

Always keep these files usable:

- `wiki/index.md` - content index of pages and short descriptions
- `wiki/log.md` - append-only chronological log
- `wiki/meta/overview.md` - scope, current thesis, and major knowledge areas

## Standard Workflows

### Ingest workflow

When asked to ingest a new source:

1. Read `wiki/index.md` and `wiki/meta/overview.md` to understand current structure.
2. Read the new source from `raw/`.
3. Create or update a page in `wiki/sources/` for that source.
4. Update relevant pages across `wiki/entities/`, `wiki/concepts/`, and `wiki/analyses/` if the source materially changes them.
5. Update `wiki/index.md` so new or changed pages remain discoverable.
6. Append a timestamped entry to `wiki/log.md`.
7. If the source introduces contradictions or uncertainty, record them explicitly.

### Query workflow

When answering a question about the wiki:

1. Read `wiki/index.md` first.
2. Open the most relevant pages.
3. Synthesize an answer grounded in the wiki and, when needed, the raw sources.
4. Update document at `wiki` if you are founding anything new.
5. If the output is durable, ask whether it should be saved under `wiki/analyses/`.
6. If saved, add it to `wiki/index.md` and append to `wiki/log.md`.

### Lint workflow

When asked to lint or health-check the wiki:

Look for:

- orphan pages with weak linking
- duplicated concepts or entities
- stale summaries superseded by newer sources
- contradictions that are not yet documented
- important topics mentioned repeatedly but lacking dedicated pages
- thin index coverage or missing log entries

Record findings in a new analysis page or directly fix small issues if asked.

## Editing Style

- Be conservative with claims that are not directly supported.
- Prefer incremental edits to broad rewrites.
- Preserve useful prior synthesis unless it is clearly outdated or contradicted.
- Keep pages concise, scannable, and link-rich.
- Add source references in page text when traceability matters.

## Log Format

Append entries to `wiki/log.md` using this format:

```md
## [YYYY-MM-DD] operation | title
- files: `wiki/path-a.md`, `wiki/path-b.md`
- notes: one or two short bullets on what changed
```

Operations usually include `ingest`, `query`, `lint`, `schema`, or `bootstrap`.

## Bootstrap Goal

Until the wiki has real content, maintain a minimal but usable structure. Do not fabricate knowledge. Empty sections should invite future updates rather than pretend the wiki is complete.

---
> Source: [woxQAQ/llmwiki](https://github.com/woxQAQ/llmwiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
