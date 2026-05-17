## obsidian-second-brain

> A personal knowledge base covering mental models, decision-making, psychology, philosophy, and more. Built from books, articles, and personal exploration, maintained by an LLM following the LLM Wiki pattern.

# Personal Knowledge Wiki — Schema

## Purpose

A personal knowledge base covering mental models, decision-making, psychology, philosophy, and more. Built from books, articles, and personal exploration, maintained by an LLM following the LLM Wiki pattern.

## Directory Structure

```
books/                          # One .md per book (frontmatter + summary + full text)
  <Book Title>.md
articles/                       # One .md per article
  <Article Title>.md
posts/                          # One .md per post
  <Post Title>.md
wiki/
  mental-models/                # Mental model pages
  (new folders added as needed) # e.g. notes/, guides/, frameworks/
index.md                        # Content catalog — updated on every ingest
log.md                          # Chronological record of operations
```

## Tags

Every page gets at least one tag in frontmatter. Tags are lowercase-kebab-case.

**Content-type tags** (what kind of page):
`mental-model`, `framework`, `note`, `guide`, `reference`

**Domain tags** (what topic area — add new ones organically):
`psychology`, `philosophy`, `relationships-and-communication`, `decision-making`, `economics`, `self-improvement`, `leadership`, `physics`, `biology`, `chemistry`, `systems-thinking`

Example: `tags: [mental-model, biology]`

## Page Conventions

### Frontmatter (YAML)

```yaml
---
tags: [content-type, domain]
source: "Book or article title"
author: "Author name"
date_ingested: YYYY-MM-DD
related: [list of [[linked pages]]]
---
```

### Linking

- Use Obsidian `[[wikilinks]]` for all cross-references
- **Keep connections tight:** limit each page to 2-3 strongest connections — ones where understanding A genuinely changes how you see B. Avoid filler links just because topics are vaguely related.
- Use `[[Page Name|display text]]` when the link text should differ from the page name

### Highlighting (Highlightr Plugin)

Use `<mark>` tags with Highlightr colors to annotate book source text. Colors indicate importance/type:

- **Yellow**: `<mark style="background: #FFF3A3A6;">text</mark>`
- **Red**: `<mark style="background: #FF5582A6;">text</mark>`
- **Blue**: `<mark style="background: #ADCCFFA6;">text</mark>`
- **Green**: `<mark style="background: #BBFABBA6;">text</mark>`
- **Pink**: `<mark style="background: #FFB8EBA6;">text</mark>`

### Mental Model Pages

Structure:
1. **One-line definition** (bold, at top)
2. **Core Idea** — 2-3 paragraph explanation
3. **Key Principles** — bullet list of the essential points
4. **Examples** — real-world applications from the source material
5. **Connections** — 2-3 strongest links to other pages (with wikilinks)
6. **Source** — which book/chapter this comes from

### Book Summary Pages

Located at `books/<Title>/summary.md`. Structure:
1. **Overview** — what the book is about
2. **Key Takeaways** — top insights
3. **Models/Concepts Covered** — list of wiki pages with wikilinks
4. **Notable Quotes** — 3-5 memorable quotes

## Workflows

### Ingest

1. Place source document in `books/<Title>/source.md`
2. Read the source fully
3. Create summary at `books/<Title>/summary.md`
4. Create/update wiki pages in the appropriate `wiki/` subfolder
5. Update `index.md` with new pages
6. Append entry to `log.md`

### Query

1. Read `index.md` to find relevant pages
2. Read those pages
3. Synthesize answer with `[[wikilinks]]` to sources
4. If the answer is substantial, offer to file it as a new wiki page

### Lint

1. Check for orphan pages (no inbound links)
2. Look for missing pages (mentioned but not created)
3. Verify cross-references are consistent
4. Flag contradictions between sources

---
> Source: [evgenirusev/obsidian-second-brain](https://github.com/evgenirusev/obsidian-second-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
