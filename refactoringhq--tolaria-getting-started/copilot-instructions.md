## tolaria-getting-started

> This is a [Tolaria](https://github.com/refactoringhq/tolaria) vault: a folder of Markdown files with YAML frontmatter forming a personal knowledge graph.

# AGENTS.md — Tolaria Vault

This is a [Tolaria](https://github.com/refactoringhq/tolaria) vault: a folder of Markdown files with YAML frontmatter forming a personal knowledge graph.

Keep edits compatible with Tolaria's current conventions. Prefer small, human-readable changes over heavy restructuring.

## Core conventions

- One Markdown note per file.
- The first H1 in the body is the preferred display title.
- Legacy `title:` frontmatter is still read as a fallback when a note has no H1. Do not add it to new notes unless you are maintaining an older file.
- Store note type in the `type:` frontmatter field.
- Most notes live at the vault root as flat `.md` files. Type definitions live in `type/`. Saved views live in `views/`.
- Any frontmatter field containing [[wikilinks]] is treated as a relationship. Common names include `Belongs to:`, `Related to:`, `Workspace:`, and custom relationship names.
- Frontmatter properties that start with `_` are usually Tolaria-managed state. Leave them alone unless the user explicitly asks for them to change.

## Notes

```yaml
---
type: Project
status: Active
icon: target
Workspace: "[[tolaria]]"
Belongs to:
  - "[[25q2]]"
Related to:
  - "[[person-luca-rossi]]"
aliases:
  - Tolaria work
url: https://example.com
---

# Ship Tolaria

Body content in Markdown.
```

## Types

Type definitions are regular notes stored in `type/`. Use `type: Type` for new ones:

```yaml
---
type: Type
icon: shapes
color: blue
sidebar label: Projects
template: |
  ## Outcome

  ## Next actions
---

# Project
```

Useful type metadata includes `icon`, `color`, `order`, `sidebar label`, `template`, `sort`, `view`, and `visible`.

## Wikilinks

- [[filename]] or [[Note Title]] — link by filename or title
- [[filename|display text]] — with custom display text
- Works in frontmatter values and Markdown body

## Views

Saved views live in `views/*.yml` and are written as YAML:

```yaml
name: Active Projects
icon: kanban
color: blue
sort: modified:desc
filters:
  all:
    - field: type
      op: equals
      value: Project
    - field: status
      op: equals
      value: Active
```

## Filenames

Use kebab-case: `my-note-title.md`. One note per file.

## What agents should do

- Create and edit notes using the frontmatter and H1 conventions above.
- Create and edit type documents in `type/`.
- Add or modify relationships without breaking existing wikilinks.
- Create and edit saved views in `views/`.
- Update `AGENTS.md` only when the user asks for vault-level guidance changes.

## What agents should avoid

- Do not infer note type from folders other than the dedicated `type/` directory for type definitions.
- Do not silently overwrite an existing custom `AGENTS.md`.
- Do not overwrite user-authored config or installation-specific app files unless the user explicitly asks.

---
> Source: [refactoringhq/tolaria-getting-started](https://github.com/refactoringhq/tolaria-getting-started) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
