## llm-wiki

> > This file is the **contract** between you and the LLM agent that maintains

# AGENTS.md — LLM-Wiki Schema & Conventions

> This file is the **contract** between you and the LLM agent that maintains
> this wiki. Read it before any ingest/query/lint operation. It tells you what
> the wiki looks like, where things go, how pages are formatted, and how to
> handle edge cases. Edit this file as conventions evolve.

## 1. Project layout

```
.
├── raw/         ← Source documents. IMMUTABLE. Read-only for the agent.
├── wiki/        ← LLM-maintained markdown. The agent owns this layer entirely.
│   ├── index.md       ← Auto-maintained catalog of every wiki page.
│   ├── log.md         ← Append-only chronological history.
│   ├── sources/       ← One summary page per ingested source.
│   ├── entities/      ← People, places, organizations, models, products.
│   ├── concepts/      ← Topics, techniques, theories, ideas.
│   └── synthesis/     ← Overview pages, comparisons, evolving theses.
└── schema/AGENTS.md   ← This file.
```

## 2. Page conventions

Every wiki page must:

1. **Start with YAML frontmatter** containing at minimum:
   ```yaml
   ---
   title: "Page Title"
   type: source | entity | concept | synthesis
   tags: [tag1, tag2]
   created: YYYY-MM-DD
   updated: YYYY-MM-DD
   sources: ["sources/source-slug.md"]   # which source pages contributed
   confidence: high | medium | low        # how confident is the synthesis
   ---
   ```

2. **Use `[[wikilinks]]`** for every cross-reference. Never plain markdown
   links for pages inside the wiki. Wikilinks are what make the Obsidian graph
   view work.

3. **Cite sources** for every non-trivial claim using a footnote-style anchor:
   `Karpathy proposed the LLM-Wiki pattern in 2026[^source-llm-wiki-gist]`,
   with the footnote linking to a `sources/` page.

4. **Be incremental.** When updating a page, preserve existing structure. Add
   sections, don't rewrite from scratch. Use `## Updates` sub-sections with
   dates if a claim is being revised.

## 3. Page types

### `sources/<slug>.md`
A summary of one ingested source. Contains: bibliographic info (title,
author, date, URL/path), 3–8 bullet key takeaways, a "Related pages" section
listing every entity/concept page touched by this source.

### `entities/<slug>.md`
A page about a single named thing (person, lab, model, product, place).
Sections: brief description, key facts, relationships (`[[wikilinks]]` to
related entities), references back to source pages.

### `concepts/<slug>.md`
A page about an idea, technique, or topic. Sections: definition, why it
matters, key examples, related concepts, source references.

### `synthesis/<slug>.md`
Overview pages, comparisons, the "evolving thesis." These are higher-order
pages that draw conclusions across multiple sources. Update these when new
sources strengthen, weaken, or contradict the thesis.

## 4. Naming rules

- **Slugs are kebab-case lowercase ASCII.** `karpathy.md`, not `Karpathy.md`
  or `andrej_karpathy.md`.
- **Use the most common/canonical name** as the slug. Disambiguate only when
  necessary (`apple-inc.md` vs `apple-fruit.md`).
- **Acronyms stay together** (`rag.md`, `llm.md`, not `r-a-g.md`).

## 5. Ingest workflow

When a new source arrives in `raw/`:

1. **Read** the full source.
2. **Identify** every named entity and significant concept.
3. **Write** `sources/<slug>.md` with the summary.
4. **For each entity:**
   - If `entities/<slug>.md` exists → append new info, update `updated` date,
     add this source to its `sources` frontmatter list.
   - Else → create the page.
5. **For each concept:** same as entities, but in `concepts/`.
6. **Update `synthesis/`** pages where this source changes the picture.
7. **Append to `log.md`:** `## [YYYY-MM-DD] ingest | <source title>` followed
   by a bullet list of pages created/updated.
8. **Update `index.md`** with any new pages.

A single source typically touches **8–15** wiki pages. That's normal and
expected.

## 6. Query workflow

When the user asks a question:

1. **Search** the wiki (BM25 + vector + rerank via QMD).
2. **Read** the top 5–10 most relevant pages in full.
3. **Synthesize** an answer in markdown.
4. **Cite every claim** with `[[wikilink]]` references.
5. **Offer to save** the answer back as a `synthesis/` page if it's a
   non-trivial new analysis.

## 7. Lint workflow

When the user runs `wiki lint`:

- **Contradictions:** Find pages making opposing claims. Flag them.
- **Orphans:** Pages with no inbound `[[wikilinks]]`. Suggest linking or
  deleting.
- **Stale claims:** Older claims contradicted by newer sources.
- **Missing pages:** Concepts mentioned 3+ times across the wiki without
  having their own page.
- **Suggested questions:** Topics where the wiki is thin and a question
  could uncover new directions.

## 8. Contradiction handling

When new source contradicts existing wiki claim:

- **Don't silently overwrite.** That destroys provenance.
- **Add an `## Updates` section** to the existing page with the date and the
  new claim, citing the new source.
- **Flag in `log.md`** with `contradiction` tag so lint can find it.
- **Update `confidence:` frontmatter** to `medium` or `low` if the
  disagreement is significant.

## 9. Frontmatter date format

Always `YYYY-MM-DD` (ISO 8601 calendar date). No times.

## 10. What the agent must never do

- ❌ **Never edit `raw/`.** Sources are immutable.
- ❌ **Never delete a wiki page** without explicit user confirmation.
- ❌ **Never overwrite existing claims silently.** Use `## Updates`.
- ❌ **Never invent citations.** If you don't know which source supports a
  claim, mark it `confidence: low` and leave the source field empty.
- ❌ **Never use plain markdown links** between wiki pages. Use `[[wikilinks]]`.

---

*This file is intentionally editable. As you and the agent figure out what
works for your domain, refine these conventions and commit the changes.*

---
> Source: [NiharShrotri/llm-wiki](https://github.com/NiharShrotri/llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
