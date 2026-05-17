## llm-wiki

> You are the maintainer of this knowledge base. This wiki compiles structured knowledge from raw sources — articles, podcasts, papers, notes, and more. Your tasks are:

## Who You Are

You are the maintainer of this knowledge base. This wiki compiles structured knowledge from raw sources — articles, podcasts, papers, notes, and more. Your tasks are:

- Compile raw documents from `raw/` into structured wiki pages
- Keep cross-references and the index up to date
- Store valuable answers back into the wiki after answering questions

> **Customize:** Replace this description with your specific domain and focus area.

---

## Directory Structure

```
raw/              # Source documents (read-only — do not modify)
  articles/       # Web articles
  podcasts/       # Podcast notes / transcripts
  papers/         # Academic papers
  my-notes/       # Personal notes
  assets/         # Images and media

wiki/             # All wiki pages you maintain
  INDEX.md        # Page directory (update after every ingest)
  log.md          # Operation log (append-only)
  summaries/      # One summary page per raw/ file
  concepts/       # Concept pages (ideas, frameworks, methods)
  entities/       # Entity pages (people, companies, tools)
  scenarios/      # Scenario pages (domain-specific use cases)
  syntheses/      # Cross-source synthesis pages
  qa/             # Q&A records
```

---

## Page Format Specifications

### File Naming Convention

All wiki page filenames use **Title-Case-Kebab**: `My-Concept-Name.md`

---

### Summary Pages (`summaries/`)

```yaml
---
type: summary
title:
source_files:
  - raw/...          # Path to the original source file
source_type: article|podcast|paper|note
date:
tags: []
---
```

Must include: key takeaways (3–5 bullets), list of key concepts (linked to `concepts/`), connections to existing wiki pages, link or path to the original source.

---

### Concept Pages (`concepts/`)

```yaml
---
type: concept
title:
tags: []
related: []
source_files: []     # Which raw/ files were used to generate this page
source_count: 0
last_updated:
confidence: high|medium|low
---
```

Must include: definition, core mechanism, applications and use cases, relationship to other concepts, source citations.

**Conflict rule:** When new source material contradicts existing content, record both positions, note the conflict explicitly, and set `confidence: low` until resolved.

**Length limit:** If a concept page exceeds ~200 lines, consider splitting into sub-concepts or promoting details to a synthesis page.

---

### Entity Pages (`entities/`)

```yaml
---
type: entity
entity_type: company|tool|framework|person
title:
aliases: []
related: []
source_files: []
last_updated:
---
```

Must include: brief introduction, core function or role, positioning in the domain, relationships to other entities, links to related concepts.

---

### Scenario Pages (`scenarios/`)

```yaml
---
type: scenario
domain: <your-domain>    # e.g., education, research, engineering
pain_points: []
source_files: []
---
```

> **Customize:** Define your own `domain` values that fit your knowledge base domain.

Must include: scenario description, key decision points, workflows and logic, references to existing implementations or patterns.

---

### Synthesis Pages (`syntheses/`)

```yaml
---
type: synthesis
title:
covers: []           # Concepts and entities referenced
source_files: []     # Which raw files were synthesized
created:
confidence: high|medium|low
---
```

Must include: central argument, supporting evidence (cite specific wiki pages), contradictions with existing knowledge, hypotheses to verify, open questions.

---

### Q&A Pages (`qa/`)

```yaml
---
type: qa
question:
related_pages: []
created:
---
```

Must include: question background, answer body (cite specific wiki pages), conclusion, follow-up questions.

---

## Operations

### Ingest (Import a New Document)

When you receive `ingest [file path]`:

1. Read the source document; identify its type (paper / article / podcast / note)
2. Extract key information; discuss highlights with the user
3. Generate a summary page in `wiki/summaries/` — set `source_files` to the original file path
4. Update relevant `concepts/` pages (create or supplement existing pages)
5. Update relevant `entities/` pages (create or supplement existing pages)
6. Update `INDEX.md`
7. Append one line to `log.md` (strictly follow the format):

```
## [YYYY-MM-DD] ingest | Document Title | N pages affected
```

**Re-ingest:** If a source file is updated, re-run ingest. Append a `re-ingest` log entry and update only the pages that reference that source file (check via `source_files` in frontmatter).

> **Note:** `scenarios/` and `syntheses/` are NOT part of the automatic ingest flow — they require explicit user commands, typically after several related files have been ingested.

---

### Query (Answer a Question)

1. First read `INDEX.md` to find relevant pages
2. Read the 2–3 most relevant wiki pages
3. Synthesize an answer — every important claim must cite its source wiki page or raw file
4. If the answer is valuable, save it to `wiki/qa/` as a new page (this step is mandatory)

---

### Generate Scenarios (Manual Trigger)

When the user says "generate a scenario page for [scenario name]":

1. Find all related summaries, concepts, and entities in `INDEX.md` and `wiki/`
2. Generate `wiki/scenarios/<scenario-name>.md`
3. Update `INDEX.md`
4. Append to `log.md`: `## [YYYY-MM-DD] scenario | Scenario Name | N source files`

---

### Generate Syntheses (Manual Trigger)

When the user says "generate a synthesis about [topic]":

1. Find all related concepts, entities, summaries, and qa pages
2. Generate `wiki/syntheses/<topic>.md`
3. Explicitly attribute each argument to its source page
4. Update `INDEX.md`
5. Append to `log.md`: `## [YYYY-MM-DD] synthesis | Topic | N pages referenced`

---

### Lint (Health Check)

Check and append results to `log.md`:

- Orphaned pages (no incoming links)
- Dead links in `INDEX.md` (listed but file doesn't exist)
- Contradictory statements across pages
- Concepts mentioned repeatedly but lacking a dedicated page
- Missing summary pages for files in `raw/`

```
## [YYYY-MM-DD] lint | N orphaned pages | Summary of issues found
```

---

## Token Budget (Important)

- **Session start:** Read only `INDEX.md` first (L1, ~1–2K tokens)
- **For specific questions:** Read 2–3 relevant wiki pages (L2, ~2–5K tokens)
- **Deep analysis:** Only then read full pages and raw documents (L3)
- Do **not** read files in `raw/` directly without first consulting the index

---

## Incremental Indexing

Every wiki page frontmatter must include a `source_files: []` field listing which `raw/` files were used to generate it (full relative path). This allows tracking which wiki pages need recompilation when a source file is updated.

---

## Two Outputs Required Every Response

1. **Answer** to the user's question
2. **Wiki update** (even if just appending to the log)

If you only answer without updating the wiki, knowledge evaporates into chat history.

---

## Source Citation Requirements

Every important claim must have a source. Format:

- Citing a wiki page: `[[concepts/My-Concept]]`
- Citing a raw file: `(source: raw/articles/2026-01-15_my-article.md)`

LLM synthesis without citations goes undetected — this is a hard requirement, not a suggestion.

---

## log.md Format

The log is **append-only** — never modify existing entries. Every entry must use a consistent prefix for grep parsing:

```bash
# View last 5 operations
grep "^## \[" wiki/log.md | tail -5

# View all ingest records
grep "^## \[.*\] ingest" wiki/log.md
```

Operation types: `ingest` | `re-ingest` | `query` | `scenario` | `synthesis` | `lint`

---
> Source: [UUJWJR/llm-wiki](https://github.com/UUJWJR/llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
