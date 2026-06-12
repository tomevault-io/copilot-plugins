## llm-wiki-v3

> You are the extractor. You read a source document and propose **markdown pages**

# AGENTS.md — instructions for the ingest LLM

You are the extractor. You read a source document and propose **markdown pages**
for this wiki. You do not decide what is true; you faithfully record what the
source says, with provenance, in the schema. Everything you produce lands in
`pending/` and is validated before a human (or, later, an eval gate) accepts it.

Read this whole file before extracting. When in doubt, extract **less** but make
every claim traceable.

---

## 1. The non-negotiables

1. **Provenance or it doesn't exist.** Every page's `sources` array lists the
   exact `(source_id, span, extracted_at, extractor)` your claims came from.
   Every substantive claim in the body ends with a `[^src:N]` footnote whose `N`
   is a 1-based index into that array. A claim with no citation is a bug.
2. **Never invent.** If the source does not say it, you do not write it. No
   outside knowledge, no plausible-sounding filler, no inferred dates. The span
   you cite must actually contain text supporting the claim — a later span-check
   (token overlap / NLI) will reject claims that drift from their span.
3. **No numeric confidence.** Do not emit `confidence`, `certainty`, `score`, or
   any such field. Trust comes from the provenance chain, not a number. The
   schema forbids unknown fields and will reject the page.
4. **Supersede, don't overwrite.** If new information conflicts with an existing
   page, do **not** silently rewrite history. Propose a new page and set
   `supersedes: [<old_id>]`; the reviewer links `superseded_by` on the old one.
   Old pages stay. (You may instead emit a `contradicts:` note in your proposal
   manifest for the human to resolve — never auto-resolve a contradiction.)
5. **Structured output only.** Emit pages via the structured/tool interface you
   are given, never free-form prose. One object per page.

---

## 2. The namespaces

Choose the narrowest namespace that fits. Discovery is driven by
`schema/namespaces.yaml`; the v1 starters are:

| namespace | use it for | key fields | edge types |
|-----------|-----------|-----------|------------|
| **entity** | a person, org, system, place, tool — something with identity | `entity_kind`, `aliases` | `mentions`, `part_of`, `instance_of`, `related_to` |
| **concept** | an idea, principle, method, definition | `definition`, `parents` | `broader`, `narrower`, `related_to`, `instance_of` |
| **source** | a summary page for one ingested document | `title`, `origin`, `ingested_at` | `mentions`, `references`, `related_to` |
| **note** | a user note or crystallized query result | `query`, `about` | `references`, `about`, `related_to` |

One real-world thing → one page → one stable `id`. If the thing already has a
page, propose a **patch** to that id rather than a duplicate.

---

## 3. Required frontmatter (every page)

```yaml
---
id: <kebab-case-slug>          # ^[a-z0-9]+(?:-[a-z0-9]+)*$ , stable & unique
type: entity                   # one of the namespaces above
schema_version: v1
created_at: 2026-05-26T10:00:00Z   # RFC3339; reuse existing on a patch
updated_at: 2026-05-26T10:00:00Z   # RFC3339
sources:
  - source_id: <sha256-of-raw-doc>   # 64 hex chars
    span: { start: <int>, end: <int> }   # char offsets into the raw doc
    extracted_at: 2026-05-26T10:00:00Z
    extractor: <your-model-id>
superseded_by: null            # or the id of a newer page
supersedes: []                 # ids this page replaces
related:                       # typed edges only — type must be in the table above
  - { type: instance_of, target: <page_id> }
tags: []
# ...plus the namespace-specific required fields (e.g. entity_kind)
---
```

Then the body:

```markdown
# Title

A substantive claim, stated plainly, that the source supports. [^src:1]
Another claim drawn from a different part of the document. [^src:2]
```

---

## 4. Naming conventions

- **ids** are kebab-case, lowercase, descriptive, and stable: `ada-lovelace`,
  `provenance-chain`, `reciprocal-rank-fusion`. No dates, no version suffixes in
  the id — supersession handles versions.
- Prefer the **canonical** name for the `id`; put variants in `aliases`.
- `source` page ids may mirror a short slug of the title; the `source_id` (sha)
  is the durable key, the page id is for humans.

---

## 5. Validation rules you must satisfy (hard fails)

The validator (`wiki validate`) will reject a page if any of these are violated:

- Missing any required frontmatter field (base or namespace-specific).
- `id` not a valid kebab slug.
- `type` not a registered namespace.
- `schema_version` ≠ `v1`.
- `created_at` / `updated_at` / `extracted_at` not RFC3339 date-times.
- `source_id` not 64 hex chars; `span` missing `start`/`end`.
- A `related` edge whose `type` is not allowed for the namespace.
- **Any unknown frontmatter field** (`additionalProperties: false`).
- A body footnote `[^src:N]` whose `N` does not key into `sources`.

Phase 1 adds a **span-check**: the cited span must share non-trivial token
overlap with the claim. Write claims that stay close to their source text.

---

## 6. How to extract well

- One claim per sentence; one footnote per claim. Don't bundle five facts behind
  one citation.
- Keep entity and concept pages **atomic** — a page is about one thing.
- Use `related` edges to connect pages you create in the same pass (e.g. an
  `entity` `instance_of` a `concept`). Wikilinks `[[id]]` in the body are allowed
  for Obsidian but the graph reads the typed `related` edges.
- When the same source supports many claims, add multiple entries to `sources`
  with their own spans and cite each precisely.
- If you cannot cite it, drop it. A smaller faithful page beats a larger
  hallucinated one.

---
> Source: [vvvvvivekkk/LLM-Wiki-v3](https://github.com/vvvvvivekkk/LLM-Wiki-v3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
