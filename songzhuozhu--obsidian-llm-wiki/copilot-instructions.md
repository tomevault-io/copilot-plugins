## obsidian-llm-wiki

> This file is the single authoritative rule entry point for the `inspool-wiki-en/` workspace and guides agents that maintain this knowledge base.

# Inspool Wiki Maintenance Rules

This file is the single authoritative rule entry point for the `inspool-wiki-en/` workspace and guides agents that maintain this knowledge base.

## Core Goal

Gradually compile scattered raw material into a structured, cross-referenced, continuously evolving English wiki.

The agent's job is not to improvise one-off answers. The job is to maintain the knowledge structure itself.

## Default Language

- The default language preference for this workspace is English.
- Unless the user explicitly requests another language, agents should write, update, and organize documents in English.
- This applies to page content, documentation, indexes, logs, and command templates.
- Raw source material may remain in its original language and does not need forced translation, but the wiki layer should default to English.

## Three-Layer Structure

### 1. `raw/`

- Stores raw source material such as web clips, articles, book excerpts, meeting notes, PDF transcripts, screenshots, and similar inputs.
- This is the factual source layer.
- The body is read-only by default.
- Only two workflow-level changes are allowed by default: moving file paths and updating a small amount of frontmatter processing metadata.
- Unless the user explicitly asks for it, do not rewrite, trim, or replace the body content under `raw/`.

### 2. `wiki/`

- Stores the structured knowledge pages maintained by the agent.
- Agents may create, update, rename, and cross-link pages here.
- When answering questions, read and synthesize from this layer first.

### 3. `AGENTS.md`

- Defines structural conventions, page rules, workflow steps, and quality standards.
- If practice reveals a better workflow, evolve this file only with user agreement.

## Directory Conventions

### `raw/`

- `raw/unprocessed/`: newly captured material waiting to be handled
- `raw/processed/`: material that completed ingest and approval
- `raw/assets/`: images, attachments, and other static assets saved from source material
- `raw/index.md`: processing-state index

### `wiki/`

- `wiki/sources/`: summary pages for individual sources
- `wiki/entities/`: pages for people, organizations, products, projects, tools, and other entities
- `wiki/concepts/`: pages for concepts, methods, frameworks, and terms
- `wiki/synthesis/`: topical overviews, comparisons, interim conclusions, and written-back Q&A results
- `wiki/meta/`: overview pages, conventions, scope notes, and maintenance metadata
- `wiki/index.md`: content index
- `wiki/log.md`: timeline log

Notes:

- Users may manually create subfolders under `wiki/concepts/`, `wiki/entities/`, and `wiki/synthesis/` for classification.
- Those subfolders are an organizational layer maintained by the user, not immutable semantic truth.
- Agents must remain compatible with arbitrary nesting inside those directories.

## Page Writing Principles

- Write in English by default unless the user explicitly asks for another language.
- Prefer clear headings and short paragraphs.
- Prefer Obsidian-style wiki links such as `[[Some Concept]]`.
- Add source-page references to important facts when possible.
- Separate facts, judgments, and speculation.
- If sources conflict, record the conflict explicitly instead of smoothing it away.

## User-Managed Classification Directories

- Users may create subfolders under `wiki/concepts/`, `wiki/entities/`, and `wiki/synthesis/`, then move pages into them.
- Treat that as normal maintenance behavior, not an error condition.
- Page type is determined by the top-level directory, such as `concepts / entities / synthesis`, not by deeper subfolder names.
- Deeper subfolders are mainly for user organization; do not overfit your reasoning to those paths.
- Unless the user explicitly asks for it, do not reorganize those classification directories on their behalf.
- When reading, updating, or linting pages, scan all subfolders recursively.

## Path Stability Rules

- Do not hard-code full file paths under `wiki/concepts/...`, `wiki/entities/...`, or `wiki/synthesis/...` as if those paths were part of the knowledge semantics.
- Prefer Obsidian wiki links in body text and frontmatter instead of bare path strings.
- If `[[Page Name]]` or another stable wiki link works, do not write a plain-text path instead.
- When a page is moved into a new classification subfolder, treat that as valid and keep operating normally.
- If classification changes leave stale path references behind, fix the references instead of asking the user to revert the structure.
- The move from `raw/unprocessed/` to `raw/processed/` is a workflow transition, not a stable semantic path.
- Therefore, source-page `raw_note` fields, body links to raw material, and any other explicit raw paths must be updated after approval.

## Evidence Nodes and Graph Principles

- Pages under `wiki/sources/` are evidence nodes in the graph.
- Pages under `wiki/entities/`, `wiki/concepts/`, and `wiki/synthesis/` are knowledge nodes.
- For key conclusions on knowledge pages, prefer links to local `sources` pages instead of direct external URLs.
- External URLs should mainly stay in raw-note frontmatter and in the "Source Information" section of source pages, not as primary graph edges.
- If you can locate a specific source-page section, prefer anchored links such as `[[sources/Some Source#Key Points]]`.
- If a conclusion comes from multiple sources, it may attach to multiple evidence nodes.
- If a conclusion has conflicting sources, record that explicitly in `contradicts` and in a body section on disagreement.

## Relationship Field Conventions

To support both Obsidian Graph and Dataview, each page should maintain:

1. explicit wiki links in the body
2. structured relationship fields in frontmatter

Recommended frontmatter fields:

- General fields:
  - `type`
  - `status`
  - `topic`
  - `updated`
- Preferred fields for source pages:
  - `raw_note`
  - `external_url`
  - `related_entities`
  - `related_concepts`
  - `related_sources`
- Preferred fields for entity, concept, and synthesis pages:
  - `supports`
  - `contradicts`
  - `related_entities`
  - `related_concepts`
  - `related_sources`
  - `related_synthesis`

Prefer local wiki links as field values, for example `[[sources/Some Source]]`.

## Citation and Link-Landing Rules

- Key conclusions, concrete facts, numbers, and controversial judgments should carry evidence links whenever possible.
- In body text, prefer evidence links such as `[[sources/...#some-section]]`.
- In frontmatter, `supports` and `contradicts` should at least point to page-level source nodes.
- If the evidence is insufficient, do not force `supports`; mark the point as "to be confirmed".
- Do not dump large numbers of bare external URLs into knowledge pages; convert them into local source-page links when possible.

## Raw Workflow Metadata

Agents may maintain a small set of workflow fields in raw-note frontmatter to reduce context pressure for later ingest runs.

Recommended fields:

- `processing_status`: `unprocessed`, `pending_review`, or `processed`
- `reviewed_at`: date of user approval
- `processed_at`: date processing completed
- `processed_into`: wiki page(s) generated from the source, preferably the source page path
- `ingest_group`: optional field for explicitly grouping related raw notes

Constraints:

- These are workflow metadata fields, not part of the source body.
- Agents may create and update these fields.
- Agents must not use them as an excuse to tamper with the source body.

## Recommended Page Structures

Not every page has to be identical, but prefer the following structures.

### Source Pages `wiki/sources/`

Recommended sections:

- Source Information
- Core Summary
- Key Points
- Evidence Excerpts
- Extractable Entities
- Extractable Concepts
- Connections to the Existing Wiki
- Open Questions
- Related Sources

### Entity Pages `wiki/entities/`

Recommended sections:

- One-Line Definition
- Background
- Key Attributes
- Supporting Evidence
- Conflicts and Disagreements
- Related Events or Views
- Related Concepts
- Related Sources
- Open Questions

### Concept Pages `wiki/concepts/`

Recommended sections:

- Definition
- Core Mechanism
- Difference From Neighboring Concepts
- Supporting Evidence
- Limits or Counterexamples
- Related Entities
- Related Sources

### Synthesis Pages `wiki/synthesis/`

Recommended sections:

- Topic Description
- Summary of Conclusions
- Supporting Evidence
- Conflicts and Disagreements
- Current Judgment
- Open Questions
- Next Suggestions
- Related Sources

## Source Citation Conventions

- Prefer appending source-page links after important conclusions, for example `See [[sources/Some Article]]`.
- If you can land on a specific section, prefer `See [[sources/Some Article#Key Points]]`.
- If a conclusion comes from multiple sources, list multiple related sources.
- If the source cannot currently be confirmed, do not present the statement as settled fact; mark it as "to be confirmed".

## Three Core Operations

### 1. Ingest

When the user asks to process new material:

1. Scan `raw/unprocessed/` first.
2. Select the next ingest unit instead of mechanically choosing a single file.
3. Read `raw/index.md`.
4. Read and extract the core information from the selected source or source batch.
5. Create or update the corresponding source summary page and attach it to the graph as an evidence node.
6. Update related entity, concept, and synthesis pages, and maintain `supports`, `contradicts`, and relevant wiki links.
7. Update `wiki/index.md`.
8. Append to `wiki/log.md`.
9. Update workflow metadata on the raw files in that ingest unit and mark them `pending_review`.
10. Update `raw/index.md`.
11. Tell the user clearly that ingest completed and approval is now required.

A single ingest run should not stop after generating one summary page. Update multiple related pages when needed.

Raw files may be marked `pending_review` only after wiki updates, log updates, and raw-index updates all succeed.

Do not move raw files into `processed/` during ingest. The move from `unprocessed/` to `processed/` must wait for user approval.

### 1b. Approve

When the user explicitly says the ingest result is correct and can be archived:

1. Scan `raw/unprocessed/` for ingest units marked `pending_review`.
2. Select the batch the user just approved; if unspecified, prefer the batch most recently moved into `pending_review`.
3. Verify again that the related wiki pages and `raw/index.md` entries exist.
4. Record the old path, new path, and `processed_into` value for each raw file in the batch.
5. Update `processing_status` to `processed` for each raw file.
6. Write `reviewed_at`.
7. Fill `processed_at` if needed.
8. Move the raw files in that ingest unit into `raw/processed/`.
9. Repair the old raw paths referenced in the wiki, including at least the source-page `raw_note` field and the body link to raw material.
10. Run a localized self-check to confirm that the old `raw/unprocessed/` paths no longer remain in `wiki/`.
11. Update `raw/index.md`.
12. Append an `approve` log entry to `wiki/log.md`.

Approval should not substantially rewrite wiki prose unless the user explicitly requests revisions during approval.

If the localized self-check in step 10 fails, do not treat the approval as complete. Report the stale paths and continue fixing them or stop with an explicit failure state.

### Ingest Unit Selection Rules

By default, the agent processes the "next batch of pending content". Determine the ingest unit with this priority:

1. If there are subfolders under `raw/unprocessed/`, treat files inside the same subfolder as one ingest unit.
2. If multiple raw files share the same `ingest_group`, treat them as one ingest unit.
3. If the user explicitly says several files should be handled together, treat them as one ingest unit.
4. If there is no grouping signal, treat one file as one ingest unit.

### Batch-Ingest Constraints

- Batch ingest is for a clearly related small group of materials.
- Recommended batch size is 2 to 5 files.
- If a batch contains more than 5 files, prefer splitting it into smaller batches to avoid context overload.
- After batch ingest, update workflow metadata for each raw file in the batch.
- When moving into `processed/`, move the full batch together to avoid half-finished states.

### 2. Query

When the user asks a question:

1. Review `wiki/index.md` first.
2. Read the relevant pages.
3. Synthesize an answer from the existing wiki.
4. For important judgments, prefer local source-page citations and anchors where appropriate.
5. If the answer has durable value, write it back into `wiki/synthesis/` when the user agrees or the scenario clearly calls for it.
6. After writing back, update `wiki/index.md` and `wiki/log.md`.

### 3. Lint

Run periodic health checks on the wiki with attention to:

- orphan pages
- strong claims without source support
- outdated conclusions invalidated by newer material but left unchanged
- frequently mentioned concepts or entities that still lack dedicated pages
- pages with clearly insufficient cross-linking
- duplicate pages or inconsistent naming
- knowledge pages that depend directly on external URLs instead of local source pages
- core pages missing relationship fields such as `supports` or `contradicts`
- pages that contain only frontmatter relationships and no explicit wiki links in the body
- stale path strings or broken directory assumptions left behind after user-driven reclassification
- mismatches between `raw/index.md` and the actual contents of `raw/unprocessed/` and `raw/processed/`
- raw notes left in `pending_review` for too long
- related raw notes that should have been processed as a group but were split apart
- ingest groups so large that they introduce too much context noise

After linting, write the result into `wiki/log.md`.

## Index Rules

`wiki/index.md` is a directory view, not a normal content page.

At minimum it should maintain:

- links to each important page
- a one-line description for each page
- optional topic grouping
- optional source counts or last-updated indicators

Before answering questions, locate relevant pages through `wiki/index.md`.

`raw/index.md` is a workflow view for queue and processed-state tracking. Do not pile workflow logs directly into `wiki/index.md`.

## Log Rules

`wiki/log.md` is a timeline, not a discussion page.

Preferred heading format for each entry:

`## [YYYY-MM-DD] action | title`

Suggested actions:

- `init`
- `ingest`
- `approve`
- `query`
- `lint`
- `refactor`

Each log body should briefly record:

- what was processed
- which pages were created or updated
- whether conflicts or follow-up work were found

## Naming Rules

- Prefer concise, stable, reusable file names.
- Do not create multiple near-synonym file names for the same concept.
- For source summary pages, it is acceptable to preserve the source topic in the file name instead of forcing date prefixes.
- If a naming migration is needed, repair links and indexes together.

## Prohibited Actions

- Do not write unverified speculation as fact.
- Do not delete raw material just to "keep things tidy".
- Do not update page bodies without also maintaining `wiki/index.md` and `wiki/log.md`.
- Do not update the wiki without also maintaining `raw/index.md` and raw-file states.
- Do not move `pending_review` raw files into `processed/` without user approval.
- Do not hide conflict information for appearance reasons.
- Do not dump unstructured blocks of chat transcript into `wiki/`.
- Do not treat external URLs as the main edges in the knowledge graph while ignoring local source pages.
- Do not misclassify user-driven subfolder changes as link errors or structural corruption.

## Default Behavior

- By default, process one ingest unit at a time; that unit may be a single file or a small batch.
- Prefer small, reviewable updates by default.
- Use Markdown by default and avoid introducing unnecessary extra systems.
- Prefer Obsidian wiki links by default and do not depend on an external database.

---
> Source: [songzhuozhu/obsidian-llm-wiki](https://github.com/songzhuozhu/obsidian-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
