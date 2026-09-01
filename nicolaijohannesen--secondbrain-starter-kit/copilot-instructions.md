## secondbrain-starter-kit

> A personal knowledge base (Obsidian-compatible vault) maintained by Claude Code. Karpathy LLM-wiki pattern: raw sources go in immutable, a compiled wiki gets built and kept current, and this file is the schema that keeps the two in sync session after session.

# {{USER_NAME}}'s Second Brain — Claude Code Instructions

A personal knowledge base (Obsidian-compatible vault) maintained by Claude Code. Karpathy LLM-wiki pattern: raw sources go in immutable, a compiled wiki gets built and kept current, and this file is the schema that keeps the two in sync session after session.

**Purpose**: {{PURPOSE_SUMMARY}}

The full statement of who this is for, what it must do well, and the milestones that mark real progress lives in [NORTHSTAR.md](NORTHSTAR.md) — read it once per session if it's been a while since the last one. This file is the operating schema; NORTHSTAR.md is the direction.

## Private-Forever (read every session — this file, not just NORTHSTAR.md)

{{PRIVACY_BOUNDARIES}}

This is the SAME boundary as [NORTHSTAR.md § What is private-forever](NORTHSTAR.md#what-is-private-forever) — restated here, not re-decided here, because this file gets read every session and NORTHSTAR.md only "once in a while." If the two ever disagree, NORTHSTAR.md is canonical and this section is stale — fix this one to match. Anything in scope here stays in `raw/` at most and never gets synthesized into a wiki page, even in general terms.

## Project Structure

```
raw/                    All source material — gitignored by default (see raw/README.md)
  inbox/                Drop zone — the only door in. Anything dropped here gets processed and routed.
  ingested/             Everything processed into the wiki, kept as provenance
    topic-sources/      Processed inbox sources, filed by kebab-case {topic}
  MANIFEST.md           Processing-status tracker — what's been ingested, what's queued

wiki/                   The vault (edit these files)
  Topics/                Conceptual/subject-matter pages
  Entities/              People, organizations, products, places
  Projects/              Your active projects and endeavors
  Sources/               One record page per ingested source
  Synthesis/             Cross-source, cross-topic analysis pages
  Personal/              Personal knowledge (health, finance, life)
  Tasks/                 The task system (see wiki/Tasks/tasks-os.md)
  Journal/               Append-only daily log, one file per day (YYYY-MM-DD.md)
  _schema.md             Page-type and frontmatter definitions — read before processing any source
  _log.md                Append-only ingestion log — update after every session

constitution/           How the assistant behaves — read at session start
  assistant-constitution.md   The operating discipline: the 5 moves, autonomy gradient, invisibility principle
  epistemic-layer.md          The 11-move thinking toolkit for rigorous inquiry
  verification.md              How claims get checked before they land in the wiki

context/                Background reading on the method itself (why this works, how to build it)
memory/                 Cross-session memory index (see memory/MEMORY.md)
scripts/                Utility scripts (parsers, ingest helpers)
```

## Wiki Page Format

Every page uses YAML frontmatter:

```yaml
---
type: topic | entity | project | source | synthesis | personal
created: YYYY-MM-DD
updated: YYYY-MM-DD
version: 1
tags:
  - topic
  - domain/science    # domain tags: pick whatever domains matter to you
aliases:
  - Alternative name
---
```

- **`version`**: bump on every substantive edit to a page (not on a typo fix). It's the cheapest signal for "has this been reworked since I last read it."
- **`updated`**: bump alongside `version`, same trigger.
- Full frontmatter spec and per-type minimums: [wiki/_schema.md](wiki/_schema.md).

### Wikilinks

- Use `[[Page Name]]` for internal links — Obsidian's graph view is driven by these.
- Link generously. If a concept mentioned in passing has or should have its own page, link it even before the page exists — Obsidian shows unresolved links distinctly, and they resolve automatically once the page is created.
- Every page should end with a `## Related` section pointing at connected pages.

### Aliases — populate richly

`aliases` are not a typo fallback. They're what keeps the graph connected as your own language varies across sessions and years. Add aliases for: common alternate terms in the literature, informal shorthand you'd naturally type, terms associated with a specific author or source, how the concept shows up in adjacent domains, and renamed versions of the same idea. Goal: any wikilink written naturally during a future session should resolve to the right page instead of creating a dead link.

## The Three Operations

1. **Ingest** — a new source arrives (conversation export, article, note, PDF). Read it, search existing pages for overlap, write or enrich the relevant Topic/Entity/Project pages, create a Source record page, cross-link everything, log it. Full workflow: the `/ingest-source` skill; page spec: [wiki/_schema.md](wiki/_schema.md).
2. **Query** — answer a question using the wiki as substrate. Read what's already documented before generating from general knowledge — the wiki is the authoritative record of your own prior thinking on a topic, not just a cache. A good answer produced during a query is itself worth filing back into the wiki as a page; don't let it evaporate in chat history.
3. **Lint** — periodic health check across the whole wiki: orphan pages (zero inbound links), stub pages, contradictions between pages, stale claims superseded by newer sources, and concepts that keep getting mentioned but never got their own page. Run every 5-10 sessions, not just at ingest.

## Inbox Pipeline

`raw/inbox/` is the only door in. Drop any raw file there; processing it moves it to `raw/ingested/topic-sources/{topic}/` when done, where `{topic}` is a kebab-case folder. Never manually move files out of the inbox — let the ingest workflow handle it so the log stays accurate and nothing silently skips processing.

**Provenance routing**: when a similarly-named file already exists in `raw/ingested/`, diff against it before reprocessing. Empty diff = duplicate, move without reprocessing. Non-empty diff = genuine new content, process it and version-suffix the filename (`_v2`) to preserve the evolution rather than overwriting.

## Journal — write continuously

`wiki/Journal/YYYY-MM-DD.md` is the time-dimension of the wiki: while pages capture *state*, the journal captures *trajectory* — decisions, corrections, what was tried and what happened. Write a timestamped entry for every substantive turn, not just at session end. **Append via Edit, never overwrite** — the journal is append-only by construction; a `Write` that replaces the whole file destroys the day's record.

The journal is the searchable cross-session memory layer: retrieve later by verbatim quote, timestamp, or concept. It narrates events and quotes you verbatim in quotation marks — it should never be written as if it were your own voice, because the two need to stay distinguishable on reread.

## Epistemic Quality

The wiki's value depends on being trustworthy, not just organized:

- **Verify load-bearing claims** — statistics, attributed quotes, and specific empirical findings get checked against external sources before they're stated as settled fact. See [constitution/verification.md](constitution/verification.md) for the two-pass discipline.
- **Label evidence types** — distinguish empirical (research-backed), expert opinion, and speculation. Don't let opinion harden into fact over time just because it's been sitting on a page for a while.
- **Flag contradictions, don't silently resolve them** — when two sources disagree, add a small section naming the contradiction, showing both claims with their sources. Don't pick a side quietly, and don't just juxtapose without flagging the tension — name it explicitly.
- **Surface assumptions** — when a source makes a confident claim, note the premise it rests on rather than inheriting it silently. Hidden premises are the primary mechanism by which a wiki drifts from accurate over time.
- **Cross-reference before writing** — search the existing wiki before generating new content on a topic it already touches. Confirm, extend, or contradict — don't duplicate.

## Session Start

Read `constitution/assistant-constitution.md` and `constitution/epistemic-layer.md` at the start of any substantive session — they're the operating discipline and the thinking-quality layer this file assumes. For a task involving contested or high-stakes claims, also read `constitution/verification.md` before writing anything.

## Model Routing

Match model tier and effort to the task (read-only lookup vs. content authoring vs. deep multi-file reasoning) rather than defaulting every sub-agent to whatever the main session is running. The routing table is [context/model-routing.md](context/model-routing.md). The general principle — cheap/fast models for retrieval and verification, mid-tier for authoring, top-tier reserved for genuinely hard reasoning.

---
> Source: [NicolaiJohannesen/SecondBrain-Starter-Kit](https://github.com/NicolaiJohannesen/SecondBrain-Starter-Kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
