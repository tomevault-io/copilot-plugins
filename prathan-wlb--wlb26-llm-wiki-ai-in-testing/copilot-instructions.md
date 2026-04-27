## wlb26-llm-wiki-ai-in-testing

> This vault is a persistent, compounding knowledge base on the domain of **Software Testing and AI**. The LLM maintains all files under `wiki/`. The human curates sources and guides exploration. Sources under `sources/` are immutable.

# LLM Wiki — Software Testing & AI

This vault is a persistent, compounding knowledge base on the domain of **Software Testing and AI**. The LLM maintains all files under `wiki/`. The human curates sources and guides exploration. Sources under `sources/` are immutable.

---

## Directory Structure

```
sources/          Raw source material — articles, papers, PDFs, transcripts. Never modified.
wiki/
  _index.md       Master index: overview, key themes, all page links organized by category.
  _log.md         Ingest log: one entry per session, date + what was processed + what changed.
  concepts/       Core ideas and techniques (e.g., mutation testing, LLM-based test gen, evals).
  tools/          Tools, frameworks, and libraries (e.g., pytest, Codium, DeepEval, Hypothesis).
  papers/         Paper and article summaries. One file per source.
  people/         Key researchers and practitioners.
  companies/      Companies and products in this space.
  synthesis/      Cross-cutting analysis: comparisons, open questions, evolving thesis pages.
```

---

## Domain Scope

This wiki covers two intersecting axes:

**Axis 1 — AI for Testing**: applying AI/ML techniques to improve software testing.
- LLM-based test generation and completion
- AI-driven fuzzing and property-based testing
- Autonomous bug detection and triage
- Self-healing test suites
- Test development
- Test execution
- Defect management
- Technical skills
- Soft skills
- Automations Testing Tools

**Axis 2 — Testing for AI**: evaluating and validating AI/ML systems.
- LLM evaluation frameworks (evals)
- Benchmarking and leaderboards
- Testing for hallucination, safety, and alignment
- Reliability and regression testing for model updates

Pages may be tagged with one or both axes using the `axis` frontmatter field.

---

## Page Conventions

### Frontmatter (all pages)

```yaml
---
title: Page Title
type: concept | tool | paper | person | company | synthesis
axis: ai-for-testing | testing-for-ai | both
tags: [comma, separated, tags]
related: [[Page One]], [[Page Two]]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

### Concepts (`wiki/concepts/`)

Structure:
1. **One-line definition** — what it is, plainly stated
2. **Why it matters** — significance in the domain
3. **How it works** — mechanism, key ideas
4. **Variants and nuances** — notable subtypes or edge cases
5. **Current state** — where the field is on this (nascent / active / mature)
6. **Open questions** — unresolved debates or gaps
7. **Related** — links to other concept, tool, and paper pages

### Tools (`wiki/tools/`)

Structure:
1. **What it does** — one-line purpose
2. **How it works** — architecture / approach
3. **Strengths**
4. **Limitations / critiques**
5. **Compared to** — explicit comparison links
6. **Who uses it** — target users, adoption signals
7. **Status** — active / archived / experimental
8. **Related** — links

### Papers (`wiki/papers/`)

Structure:
1. **Citation** — authors, venue, year, URL if available
2. **Core claim** — the central thesis in 1-2 sentences
3. **Key contributions** — bullet list
4. **Method** — how they did it
5. **Results** — key numbers or findings
6. **Limitations** — what they acknowledge or what's missing
7. **My take** — synthesis commentary (where does this fit, is it credible, what does it change)
8. **Related** — links

### People (`wiki/people/`)

Structure:
1. **Role / affiliation**
2. **Known for** — key contributions or positions
3. **Key works** — links to paper pages
4. **Perspective** — any notable stance or school of thought
5. **Related** — links

### Companies (`wiki/companies/`)

Structure:
1. **What they do** — one-line
2. **Product(s)** — links to tool pages where applicable
3. **Axis** — which side of the domain they're on
4. **Stage / status** — startup / public / acquired / defunct
5. **Notable** — funding, customers, controversies
6. **Related** — links

### Synthesis (`wiki/synthesis/`)

Free-form. Used for:
- Comparison tables (e.g., "LLM test gen tools compared")
- Open questions and research gaps
- The evolving thesis on how this domain is developing
- Answers to complex questions worth preserving

---

## Workflows

### Ingest a Source

When the user says "ingest [source]" or drops a file in `sources/`:

1. **Read** the source fully.
2. **Discuss** with the user: What are the 3-5 key takeaways? Anything surprising or that contradicts existing wiki pages?
3. **Write a paper/article summary page** in `wiki/papers/` (filename: slugified title).
4. **Update or create concept pages** for any significant idea touched. Check existing pages first — update rather than duplicate.
5. **Update or create tool/company/person pages** for any entities mentioned significantly.
6. **Update `wiki/_index.md`** — add new pages to the catalog tables with a one-line summary; update Key Themes if the source shifts the overall picture; increment `page_count` and `source_count` in frontmatter.
7. **Append to `wiki/_log.md`** — use the format `## [YYYY-MM-DD] ingest | Source Title`, then list pages created/updated and any contradictions found.

Aim to touch all relevant pages, not just create a new summary page. A good ingest updates 5-15 pages.

### Answer a Question

When the user asks a question about the domain:

1. **Read `wiki/_index.md` first** — it's the catalog. Identify which pages are relevant from the table entries and key themes.
2. Read the relevant pages identified.
3. Synthesize an answer with citations to wiki pages (`[[page name]]`).
4. Ask: "Should I save this as a synthesis page?" If yes, write it to `wiki/synthesis/` and add it to the index.

### Lint the Wiki

When the user says "lint" or "health check":

1. Scan all wiki pages.
2. Report:
   - Contradictions between pages
   - Orphan pages (no inbound links from `_index.md` or other pages)
   - Concepts mentioned in text but lacking their own page
   - Missing cross-references that should exist
   - Stale "current state" claims that newer sources may have superseded
3. Suggest 3-5 questions worth investigating or sources worth finding.

---

## Style Rules

- Write in clear, direct prose. No filler. No hedging without cause.
- Wikilinks (`[[Page Name]]`) for all cross-references. Use the exact filename without extension.
- If a concept appears in a page but doesn't have its own page yet, create a stub.
- Never delete content from a page without noting what was removed and why in the log.
- Contradictions: don't silently overwrite. Add a note: `> **Contradicts [[Other Page]]**: ...`
- Dates in frontmatter: always `YYYY-MM-DD`.
- Keep page filenames lowercase, hyphenated (e.g., `mutation-testing.md`, `codium-ai.md`).
- **Frontmatter discipline**: consistent YAML frontmatter enables Dataview queries. Always include `type`, `axis`, `tags`, `created`, `updated`. The Dataview plugin can then generate dynamic tables like "all tools tagged `llm` sorted by updated date."

## Log Format Reference

```
## [YYYY-MM-DD] ingest | Source Title
## [YYYY-MM-DD] query  | Question asked
## [YYYY-MM-DD] lint   | Health check
## [YYYY-MM-DD] init   | Setup event
```

Quick grep examples:
```bash
grep "^## \[" wiki/_log.md | head -5        # last 5 entries
grep "^## \[.*\] ingest" wiki/_log.md       # all ingests
grep "^## \[.*\] query" wiki/_log.md        # all queries answered
```

---

## Current Focus Areas

_(Update this section as the wiki matures)_

- Getting started — bootstrapping core concept pages for foundational ideas in both axes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Prathan-WLB) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
