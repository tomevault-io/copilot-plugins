## llm-wiki-vjack

> This is an **LLM Wiki** — the pattern from Andrej Karpathy's

# llm-wiki — schema & operating rules

This is an **LLM Wiki** — the pattern from Andrej Karpathy's
[llm-wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) (April 2026),
adapted for a master-data / data-quality team. Canon kept: three layers, agent-maintained pages,
index + append-only log, lint. Adaptations: the index is called **`MAP.md`** (with a scaling
rule), plus two extra content spaces — `wiki/directory/` and `snippets/`. Per Karpathy: *"you
and the LLM co-evolve this over time as you figure out what works for your domain."*

A compounding knowledge base: knowledge is **compiled once and kept current** — never re-derived
from raw sources on every query. The human curates sources and asks questions; **the agent
writes and maintains all of the wiki** — the tedious part is the bookkeeping, and that is the
agent's job.

**🧭 Navigation contract: read [`MAP.md`](MAP.md) first — it is the GPS.** It tells you what
exists and where; its summaries and tags let you decide relevance *without* opening pages.
Never scan the tree when the map can route you.

**💰 Token economy (hard rule — this runs on metered API credits):** cheap before expensive,
always. Map → summaries → only the pages that matter → follow `[[links]]` on the shortest
path. Full-text search over the tree is the last resort, not the first move. Keep this file
and the map lean; volume lives in pages.

**🌐 English only (hard rule):** this repo is English, end to end — pages, snippets, `MAP.md`,
`log.md`, templates, commit messages, PR titles and descriptions, and **the agent's own
responses while working in this repo**. It is built to be shared with an English-speaking team;
no other language appears anywhere in the repo or the working session, whatever the operator's
default language is.

---

## Layers

1. **`raw/`** — original sources. Human-owned. **Immutable, append-only** (regime: `raw/README.md`).
2. **`wiki/` + `snippets/`** — the knowledge. Agent-maintained, written continuously.
3. **`CLAUDE.md` + `MAP.md`** — schema & GPS. Human-curated; change deliberately.

## Principles

1. **Persistence over retrieval** — a compounding artifact, not a search cache.
2. **LLM-maintained** — the agent does the bookkeeping (links, updates, consistency); the human curates.
3. **Source immutability** — `raw/` never changes; only the knowledge layer evolves.
4. **Compounding** — one ingest should enrich several existing pages, not just add one.
5. **Continuous synthesis** — a reusable answer becomes a page, so it is never re-derived.
6. **Single source of truth** — one fact lives on exactly ONE page; everything else links to
   it with `[[page-name]]`. Never copy content between pages — copies desync and burn tokens.
7. **Grow as a process** — sources arrive incrementally (documents, links, snippets), not as a
   one-shot dump. The catalog is built to scale (see MAP scaling rule).
8. **Human merge rule** — the agent may create branches, commit work, and push it to GitHub;
   the final merge into `main` must be done explicitly by a human reviewer on GitHub.

## Working style

- Human input often arrives as a **stream of thought** carrying several distinct points. FIRST
  break it into an explicit bullet list of every point/hook, THEN address each one — nothing
  gets dropped, nothing half-answered. Unclear points → ask, one at a time.
- Useful beats perfect. Ship the working version; polish where it pays.

## Session rituals

- **▶️ START:** run **`/start`** — git sync (pulls merged PRs), map, log tail, flags anything
  waiting at the door (uningested sources, open PRs) — know the state before touching anything.
- **⏹️ STOP:** run **`/wrap`** before ending a session — touched pages updated, MAP in sync,
  log appended, work committed. An unclosed session leaves the wiki lying about itself.
- **🔍 Periodically:** run **`/scout`** — Karpathy's failure mode is *drift* (cross-references
  silently going stale); the lint pass is not optional.

## Page conventions

- Filename = kebab-case of the title; stable (pages link to it).
- Header block (template: [`wiki/_templates/note.md`](wiki/_templates/note.md)):
  `Summary:` one load-bearing sentence · `Tags:` few, reused (registry in MAP) ·
  `Type:` concept | entity | directory-entry | snippet | source-summary · `Updated:` `YYYY-MM-DD`.
- Cross-link generously with `[[wiki-links]]`; a link to a not-yet-written page is a valid TODO.
- Contradiction between sources → do NOT silently overwrite; add a `> ⚠️ Contradiction:` note
  with both claims and sources, let the human resolve.
- Citations: claims trace to sources (a `raw/` file or an external reference in `Sources`).
- **Agnostic examples:** demo/mockup content uses generic placeholders — no real company,
  system, or person names.
- **Self-contained repo:** references only its own files; never reaches into any other system.

## Content spaces

- **`wiki/concepts/` · `wiki/entities/`** — knowledge pages (ideas/frameworks · tools/systems/orgs).
- **`wiki/summaries/`** — one page per ingested `raw/` source: key takeaways, linking out to
  every page the source fed. (Created on first ingest.)
- **`wiki/directory/`** — *the where-things-live catalog*: entries that hold no knowledge
  themselves but point to external resources (docs, sites, repos, reports) — what it is, where,
  owner, how to access. This space is expected to grow large; it scales via the MAP rule.
- **`snippets/`** — canonical, known-good code samples in the house conventions, one file per
  snippet with a header (What / When / Convention) so agents can reuse them verbatim.

## Workflows

**Ingest** (a source or a reference): run **`/ingest`** — source secured in `raw/` (or an
external link cataloged as a directory entry) → takeaways discussed with the human → summary
page → the pages it enriches updated/created (expect several, not one) → cross-links both
contradictions flagged → `MAP.md` + `log.md` synced → commit and push to a branch → **PR opened;
the human reviews and clicks merge on GitHub**. The agent does not merge its own PRs; GitHub
merge is the final approval step.

**Answer a query:** MAP → triage by summaries/tags → read only what matters → answer *with
citations to pages*. No page supports a claim → say so (that's a gap, not a thing to invent).
Reusable answer → capture as a page (principle 5).

**Maintain:** `/wrap` closes every session; `/scout` runs the periodic lint (checklist lives
in the skill).

## Skills

- **`/start`** — session opening brief: [`.claude/skills/start/SKILL.md`](.claude/skills/start/SKILL.md)
- **`/ingest`** — a source enters the wiki (ends in a PR for human merge): [`.claude/skills/ingest/SKILL.md`](.claude/skills/ingest/SKILL.md)
- **`/wrap`** — deterministic session close: [`.claude/skills/wrap/SKILL.md`](.claude/skills/wrap/SKILL.md)
- **`/scout`** — read-only integrity sweep: [`.claude/skills/scout/SKILL.md`](.claude/skills/scout/SKILL.md)

---
> Source: [Daltonist92/llm-wiki-vJack](https://github.com/Daltonist92/llm-wiki-vJack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
