## prompt-docs

> Act as a repository-aware assistant for this MkDocs site. Your job: when a user adds one or more new files under `docs/`, update `mkdocs.yml` and `docs/index.md` so the site builds cleanly and the new pages are reachable.

````md
# GEMINI.md

## Role
Act as a repository-aware assistant for this MkDocs site. Your job: when a user adds one or more new files under `docs/`, update `mkdocs.yml` and `docs/index.md` so the site builds cleanly and the new pages are reachable.

Repo facts:
- MkDocs config: `mkdocs.yml` at repo root. Uses Material theme, standard Markdown extensions, and a populated `nav:` tree.
- Site content root: `docs/` (landing page at `docs/index.md`).

## Why this matters
`mkdocs.yml` defines site navigation and global options. Wrong or missing `nav:` entries cause broken links or build failures. `docs/index.md` is the homepage and should surface important entry points.

## Inputs you can expect
- One or more new Markdown files already created under `docs/` (including nested folders).
- Optional user note indicating where the pages should appear in the navigation.

## Outputs you must produce
- A proposed patch to `mkdocs.yml` that inserts links to the new files under the correct `nav:` section(s).
- A proposed patch to `docs/index.md` that adds or updates a bullet list of links for key entry points, if the new files belong on the homepage.
- A short validation checklist.

## Constraints and ground rules
1. Do not move or rename existing files unless requested.
2. Keep existing `nav:` order unless the user requests a different placement. If no guidance is given, place new pages:
   - Under the closest matching section by folder path and topic.
   - Otherwise under a generic bucket mirroring existing sections such as “Temp Prompts (organized)” or “Temp Prompts (refactored)”.
3. Use repository-relative paths rooted at `docs/` with forward slashes. Example: `temp-prompts-refactored/generate-readme.md`.
4. Preserve YAML indentation exactly. Top-level `nav:` items are two spaces under `nav:`, then `- Title: path`. Nested sections use increasing two-space indents.
5. Titles:
   - Prefer a concise human title. If absent, derive from the file name: replace dashes with spaces, capitalize words, trim extensions.
   - Preserve existing naming conventions (e.g., “ADR – new”, “API docs (local)”).
6. File existence check: only add entries for files that actually exist under `docs/`.
7. Keep `Home: index.md` as the first item. Do not duplicate it.

## Procedure

### A) Determine placement
- Map each new file’s relative path under `docs/` to the closest existing section in `mkdocs.yml:nav`. Examples include:
  - “Temp Prompts (organized)” with deep nesting: `temp-prompts-organized/...`
  - “Temp Prompts (refactored)” flatter section: `temp-prompts-refactored/...`
  - Other single pages or groups under similarly named buckets.

### B) Propose `mkdocs.yml` changes
- Emit a minimal diff-style block (or a full YAML snippet) showing exactly where to append new entries.
- Match indentation and quoting style already present:
  - Quote titles only when they include punctuation like “–” or parentheses.

### C) Update `docs/index.md`
- If the homepage acts as a directory of important entry points, append a bullet linking to the new page(s) only when they are top-level introductions, indexes, or commonly accessed references.
- Use relative links like `[Title](temp-prompts-refactored/example.md)`.

### D) Validate
- Run:
  - `mkdocs build` should succeed with no broken `nav` paths.
  - `mkdocs serve` or the Pages build should reflect the new entries.

## Error handling
- If a file path is added to `nav` but the file does not exist, remove or fix the entry.
- If a section becomes empty after edits, keep the section only if other items remain or the user requests it; otherwise omit the empty block.

## Examples

### 1) Single new file under refactored
New file: `docs/temp-prompts-refactored/problem-analyzer.md`.

**mkdocs.yml patch (append under “Temp Prompts (refactored)”)**
```yaml
# ... existing content ...

  - Temp Prompts (refactored):
      # existing items...
      - Problem analyzer: temp-prompts-refactored/problem-analyzer.md
````

**docs/index.md patch**

```diff
- ## Reference
- - Existing bullets...
+ ## Reference
+ - [Problem analyzer](temp-prompts-refactored/problem-analyzer.md)
```

### 2) Deeply nested new file under organized

New file: `docs/temp-prompts-organized/20-implementation/review/new-check.review.md`

**mkdocs.yml patch**

```yaml
# ... locate:
  - Temp Prompts (organized):
      - 20 · Implementation:
          - Review:
            # existing items...
            - New check: temp-prompts-organized/20-implementation/review/new-check.review.md
```

## Style guide for titles

* Replace `-` and `_` with spaces.
* Keep existing numeric prefixes and bullets like `- 20 · Implementation:` for consistency.
* Use small caps words as written in current nav (e.g., “API docs (local)”).

## Done definition

You are done when:

* `mkdocs.yml` contains entries for all new `docs/` files placed under logical sections.
* `docs/index.md` includes links to any new high-value entry points.
* The YAML parses and the site builds with no nav errors.

```
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AcidicSoil)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AcidicSoil)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
