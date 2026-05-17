## claude-wiki-research-skills

> Read at start of every session before any wiki work.

# Wiki Operating Schema

Read at start of every session before any wiki work.

## Purpose

LLM-maintained research wiki for an academic researcher. Configure the research topics below for your domain. Default topics:
- Learning theory (decolonial, mycelial, challenge-based, boundary crossing)
- Sustainability transitions + higher education
- Systems thinking + wicked problems
- Education design research (EDR)
- AI in education and SME contexts
- Lifelong Learning

Wiki feeds papers, research projects, and a future book. Not a note-taking system — a compiled, synthesised knowledge base.

## Configuration

Set these in your local copy before using the skills:

```
WIKI_ROOT: /path/to/your/wiki/
RAW_ROOT:  /path/to/your/raw/
WIKI_LANGUAGE: en
RESEARCHER_NAME: Your Name
INSTITUTION: Your Institution
```

## Language Policy

**All wiki pages in English.** Raw sources may be in any language. Summarise non-English sources in English. Quote key terms in the original language where no clean English equivalent exists, with translation in parentheses.

## Directory Layout

```
wiki/
├── CLAUDE.md          ← this file (operating schema)
├── index.md           ← master catalog of all wiki pages
├── log.md             ← append-only session log
├── overview.md        ← high-level synthesis of entire research landscape
├── concepts/          ← one page per theoretical concept
├── thinkers/          ← key scholars: position, key works, debates
├── themes/            ← synthetic pages that map directly to papers/book chapters
├── tensions/          ← explicit contradictions, open questions, unresolved debates
├── sources/           ← one summary page per ingested source
├── outputs/           ← paper outlines, chapter structures, writing plans
└── fleeting/          ← FUNGI notes: atomic, claim-shaped notes at any maturity stage

raw/
├── articles/          ← source PDFs, clipped markdown articles
├── books/             ← book chapters, excerpts
├── podcasts/          ← podcast notes, transcripts
└── legacy/            ← archived old notes (read-only, never modify)
```

**Raw sources immutable.** Never edit `raw/`. Read from it; write only to `wiki/`.

## Page Formats

### concepts/[concept-name].md
```markdown
---
type: concept
title: [[Full Concept Name]]
aliases: [other names for this concept]
related_concepts: 
- "[[concept-a]]"
- "[[concept-b]]"
key_thinkers:
- "[[thinkers/thinker-a]]"
themes:
- "[[themes/theme-a]]"
sources:
- "[[sources/source-a]]"
last_updated: YYYY-MM-DD
---

# [Concept Name]

## Definition
[2–4 sentence working definition. Synthesized, not copied.]

## Core Elements
[Numbered or bulleted breakdown of key components]

## Theoretical Lineage
[Where does this concept come from? Who developed it? How has it evolved?]

## Relevance to This Research
[Why does this concept matter for the researcher's work specifically?]

## In Tension With
[Concepts or positions this concept conflicts with — link to tensions/ pages]

## Key Sources
[Annotated list of 3–6 most important sources for this concept]
```

### thinkers/[lastname-firstname].md
```markdown
---
type: thinker
name: Full Name
institution: Current or last known institution
field: [discipline, subdiscipline]
related_concepts:
- "[[concepts/concept-a]]"
key_thinkers:
- "[[thinkers/thinker-b]]"
sources:
- "[[sources/slug]]"
themes:
- "[[themes/theme-a]]"
last_updated: YYYY-MM-DD
---

# [Full Name]

## Position in Brief
[2–3 sentences: what is this thinker's central argument or contribution?]

## Key Works
[Annotated list of most important works. Mark ingested works with source link.]

## Core Concepts
[Link to concept pages this thinker developed or heavily influenced]

## Relation to This Research
[How does this thinker's work connect to the research agenda?]

## Debates and Critiques
[Who disagrees with them and why? What are the limits of their work?]
```

### themes/[theme-name].md
```markdown
---
type: theme
title: [Theme Name]
related_themes:
- "[[themes/theme-b]]"
key_concepts:
- "[[concepts/concept-a]]"
key_thinkers:
- "[[thinkers/thinker-a]]"
feeds_into: "Paper title or Book: Chapter X"
source_count: N
last_updated: YYYY-MM-DD
---

# [Theme Name]

## Research Question / Thesis Direction
[The central question or argument this theme is building toward]

## Current Synthesis
[The substantive synthesis — most important section. What do we know? What is the argument?]

## Key Evidence and Sources
[Annotated list of sources that support the synthesis]

## Gaps and What's Missing
[What would strengthen this? What sources are needed?]

## Connections to Other Themes
[How this theme intersects with others]
```

### tensions/[tension-name].md
```markdown
---
type: tension
title: [Tension Name]
concepts_involved:
- "[[concepts/concept-a]]"
- "[[concepts/concept-b]]"
thinkers_involved:
- "[[thinkers/thinker-a]]"
themes_involved:
- "[[themes/theme-a]]"
status: open | partially_resolved | resolved
last_updated: YYYY-MM-DD
---

# [Tension Name]

## The Tension
[Plain-language statement of the contradiction or unresolved question]

## Position A
[Statement of one side, with key sources]

## Position B
[Statement of the other side, with key sources]

## Current Assessment
[Where does the researcher's thinking sit? What would resolve this?]

## Why This Matters
[Implications for the research]
```

### sources/[author-year-shorttitle].md
```markdown
---
type: source
title: "[Full Title]"
author: Lastname, Firstname; Lastname2, Firstname2
year: YYYY
journal: Journal Name
volume: N
issue: N
pages: NNN–NNN
doi: https://doi.org/...
pmcid: PMCXXXXXXX
read: true
read_date: YYYY-MM-DD
file: "raw/articles/filename.pdf"
feeds_into:
- "[[concepts/relevant-concept]]"
- "[[fleeting/relevant-note]]"
---

# [Short Citation]

## In Brief
[2–3 sentences: what the source does and why it matters for this research]

## Core Theoretical Framework
[Key concepts, mechanisms, claims. Use subheadings. Quote directly where precise.]

## Relation to Research
[Explicit connections to the wiki's conceptual vocabulary]

## Tensions with Other Wiki Frameworks
[Compare/contrast with other wiki sources or concepts]

## Technical Note
[Flag parts not relevant to this research programme, if applicable]

## Pages Updated
[Which wiki pages were touched when this was ingested]
```

### fleeting/[claim-slug].md
```markdown
---
type: fleeting
title: "[claim-shaped title — one sentence]"
date: YYYY-MM-DD
status: seedling | developing | mature
connections:
- "[[concepts/page-a]]"
- "[[sources/source-b]]"
confidence: H | M | L
promoted: false
promoted_to:
---

# [Claim title]

## 1. Claim
[One sentence, identical to the title. Claim-shaped, not topic-shaped.]

## 2. Frame
[The theoretical lens. Name it explicitly.]

## 3. Substrate
[APA references that ground the claim.]

## 4. Connections
[≥2 links to existing wiki pages. State the connection explicitly.]

## 5. Generative question
[What new inquiry does this claim open?]

## 6. Counter-argument
[The strongest objection. If you cannot state it, the claim is not ready.]

## 7. Confidence
**H / M / L** — [one sentence explaining why]

## 8. Open threads
[What does this note need to mature?]
```

**FUNGI filename convention**: slugify the claim, not a topic label. `scalable-knowledge-destroys-relational-context.md`, not `tsing-notes.md`.

**Status progression**:
- `seedling` — claim stated, frame named, ≥1 source; counter-argument may be thin
- `developing` — counter-argument addressed, ≥2 connections confirmed, generative question sharpened
- `mature` — ready to promote: claim can stand in paper argument without further hedging

**Promotion rule**: mature fleeting note → incorporate into concept, tension, theme, or output page. Set `promoted: true` and `promoted_to: [[target-page]]`. Do not delete the note — it is the audit trail.

---

## Operations

### Ingest a new source
1. User provides source (drops file in `raw/`, pastes text, or gives URL)
2. Read source fully
3. Write `sources/` page
4. Update or create relevant `concepts/`, `thinkers/`, `themes/` pages
5. Update or create `tensions/` pages if contradictions surface
6. Update `index.md`
7. Append to `log.md`
8. Single source typically touches 5–15 wiki pages

### Create a FUNGI note
1. User provides raw idea, quote, observation, or half-formed claim
2. Distil to single claim sentence (claim-shaped, not topic-shaped)
3. Identify theoretical frame it belongs to
4. Identify ≥2 existing wiki pages it connects to
5. Write `fleeting/[claim-slug].md` using FUNGI schema
6. Set `status: seedling`
7. Append brief entry to `log.md`
8. Add to Fleeting Notes section of `index.md`
9. Do **not** update concept/theme pages yet — that happens at promotion

### Promote a FUNGI note
1. User signals note is ready (status: mature, or explicitly asked)
2. Identify target wiki page(s) the claim should be incorporated into
3. Integrate claim into those pages (concept, tension, theme, or output)
4. Set `promoted: true` and `promoted_to: [[target-page]]` in frontmatter
5. Update `index.md` — mark entry as promoted with pointer to target
6. Append to `log.md`

### Lint the wiki (use `/wiki-link-check`)
**Standard checks (every session):**
1. **A1 — Orphan Detector**: scan all pages for zero inbound wikilinks; group by folder; distinguish expected orphans (fresh fleeting) from structural gaps (unlinked tensions, sources, thinkers)
2. **A2 — Frontmatter Completeness**: check each page type against schema above; flag missing `type`, `last_updated`, YAML list errors
3. **A4 — Gap-to-Ingest Priority List**: identify sources referenced ≥5× and thinkers referenced ≥3× without their own page; rank by load-bearing status

**Rotating analytical checks (≥1 per session):**
- **B1 — Fruiting Body Detector**: flag pages where high connectivity suppresses tension; ask what it is hiding
- **B2 — Lateral Connector**: surface structural homologs across clusters lacking a wikilink or tension page
- **B3 — Asabiyyah Check**: for each framework, state what social bond sustains it and what institutional condition dissolves it
- **B4 — Dynasty Cycle Audit**: assign Ibn Khaldun generation (1/2/3) to mature frameworks; name the next decay warning sign
- **B5 — Simulacra Order Check**: assign Baudrillard order (1–4) to institutional concepts

### Build a paper outline
1. User names paper or research question
2. Read relevant `themes/`, `tensions/`, `concepts/` pages
3. Propose structure as new `outputs/[paper-name].md` page
4. Identify what is ready to write, what needs more sources, what needs resolution
5. Update `index.md`

## Index Conventions

`index.md` has sections: Concepts, Thinkers, Themes & Tensions, Sources, Outputs, Fleeting. Each entry is one line:
```
- [[page-name]] — one-line description
```

Keep it scannable. This is how Claude navigates the wiki at query time.

## Log Conventions

Each log entry:
```
## [YYYY-MM-DD] operation | title
```
Operations: `ingest`, `query`, `lint`, `outline`, `founding-pass`, `session`

## Maintenance Rules

- Never leave concept page without ≥1 `related_concepts` link
- Never leave thinker page without ≥1 `key_concepts` link
- Every `themes/` page must have `feeds_into` field naming specific paper or book chapter
- When sources contradict each other, create `tensions/` page — do not silently pick one
- When adding new page, always update `index.md` in same session
- Prefer updating existing pages over creating new ones when content overlaps
- `overview.md` updated at least every 5 ingests or when major synthesis shift occurs

## On the Research Agenda

Adapt this section to your own research programme. The researcher is working toward:
1. **Papers** on [your research topics]
2. **A book** — [scope TBD]
3. **Research projects** — [institutional context]

When building theme pages or paper outlines: keep both international academic audience and local institutional context in mind. Work should bridge theory and practice.

---
> Source: [FBoschman/claude-wiki-research-skills](https://github.com/FBoschman/claude-wiki-research-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
