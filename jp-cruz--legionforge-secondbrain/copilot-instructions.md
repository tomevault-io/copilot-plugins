## legionforge-secondbrain

> > This file is the schema for LegionForge SecondBrain (LF2B).

# LF2B — AI Operating Instructions

> This file is the schema for LegionForge SecondBrain (LF2B).
> It tells the AI agent how to behave when working in this repository.
> Human curates sources. AI maintains the wiki. Do not conflate these roles.

---

## Project Identity

- **Name:** LegionForge SecondBrain (LF2B)
- **Owner:** JP Cruz / LegionForge
- **Purpose:** Sovereign, portable, AI-maintained personal knowledge base
- **Core constraint:** All wiki content must be traceable to a raw source. No hallucinated facts in the wiki.

---

## Directory Roles

| Directory | Owner | Rule |
|-----------|-------|------|
| `raw/` | Human | Immutable. AI reads only, never writes or modifies. |
| `wiki/` | AI | AI creates and updates. Human reads and reviews. |
| `src/` | Human + AI | Code — treat like any software project. |
| `docs/` | Human + AI | Architecture decisions, guides. |

---

## Wiki Conventions

### File naming
- Concept pages: `wiki/concepts/<slug>.md`
- Entity pages: `wiki/entities/<slug>.md`  
- Topic pages: `wiki/topics/<slug>.md`
- Synthesis pages: `wiki/syntheses/<slug>.md`
- Index: `wiki/_index.md`

### Page structure
Every wiki page must include:
```markdown
---
title: <Page Title>
type: concept | entity | topic | synthesis
sources: [<raw file paths that informed this page>]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

### Cross-references
Use `[[page-slug]]` wikilink format for internal references. Every entity mentioned in a page should link to its entity page if one exists.

### Contradictions
When a new source contradicts existing wiki content, do NOT silently overwrite. Add a `## Contradictions` section noting the conflict and both sources. Flag with `#needs-review`.

### Freshness
Add `#stale` tag to any claim older than 6 months that has not been re-verified by a newer source.

---

## Ingestion Workflow

When a user adds a file to `raw/` and runs `/wiki-ingest <file>`:

1. Read the source file
2. Discuss key takeaways with the user (ask 2-3 clarifying questions)
3. Write a summary page in `wiki/topics/` or update an existing one
4. Update or create entity pages for any people, orgs, or tools mentioned
5. Update concept pages if new nuance is introduced
6. Update `wiki/_index.md` with the new pages
7. Log the ingestion action at the bottom of `wiki/_index.md`
8. Do NOT modify `raw/` files

---

## Query Workflow

When a user asks a question:

1. Search `wiki/_index.md` for relevant pages
2. Read relevant pages
3. Synthesize answer from wiki content only (do not hallucinate beyond wiki)
4. If the answer is not in the wiki, say so — do not guess
5. If the answer is valuable enough to persist, offer to write it as a new synthesis page

---

## Lint Workflow

When `/wiki-lint` is run:

- Check for orphan pages (not linked from `_index.md`)
- Check for pages missing required frontmatter
- Check for `#needs-review` and `#stale` flags
- Check for broken wikilinks
- Report findings — do not auto-fix without user confirmation

---

## LLM Provider Notes

- Default: Ollama (local). Prefer local over cloud.
- If using a cloud provider, do not send `raw/` content that the user has marked as `#private`
- Model switching does not affect wiki content — the wiki is model-agnostic

---

## Security Rules

- Never commit API keys, LAN IPs, internal hostnames, or SSH details
- `raw/` files marked `#private` must never be sent to cloud LLM APIs
- The wiki is designed to be publicly shareable; raw sources are not
- Run the pre-commit scrub from global CLAUDE.md before any push

---

## Scope Discipline

This is a v0.1 project. Rules:
- Build ingest and query before agents
- Build local LLM support before multimodal
- Ship working core before adding sync protocols
- "Good enough and shipped" beats "perfect and parked"

---
> Source: [jp-cruz/LegionForge-SecondBrain](https://github.com/jp-cruz/LegionForge-SecondBrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
