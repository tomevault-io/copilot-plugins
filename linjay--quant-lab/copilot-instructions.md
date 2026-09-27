## quant-lab

> These instructions apply to the whole repository.

# quant-lab agent instructions

These instructions apply to the whole repository.

## Project wiki

This repository uses a Karpathy-style LLM Wiki: versioned project sources are
the source of truth, while `wiki/` is an LLM-maintained, cross-linked
knowledge layer compiled from those sources.

When a task depends on understanding the project:

1. Read `wiki/index.md`.
2. Read only the relevant wiki pages.
3. Follow their citations into source files when exact behavior, current
   values, or historical wording matters.
4. If a source contradicts the wiki, trust the source, update the affected
   wiki pages, and record the correction in `wiki/log.md`.

The precedence order is:

1. executable code, tests, configuration, and tracked data contracts;
2. current canonical project documents;
3. compiled wiki pages;
4. generated or ignored runtime artifacts.

The wiki summarizes evidence. It never overrides executable behavior or turns
research output into investment advice.

## Source layer

`wiki/raw/index.md` is the source manifest. Existing repository files stay in
their canonical locations and are immutable *within an ingest operation*.
Git history supplies snapshot/version provenance, so do not duplicate tracked
project documents under `wiki/raw/`.

External material that is not already versioned may be added under
`wiki/raw/inbox/` with a date, origin URL, and retrieval note. Never silently
edit an ingested external source; add a new version instead.

## Wiki ownership and page schema

The LLM owns files under `wiki/`, except the source material under
`wiki/raw/inbox/`. Humans normally review the wiki rather than hand-maintain
it.

Compiled pages use this frontmatter:

```yaml
---
title: Human-readable title
summary: One sentence that helps both humans and agents decide whether to read
status: current | historical | review
updated: YYYY-MM-DD
sources:
  - repository/relative/path
---
```

Keep pages concept-oriented rather than mirroring one source file per page.
Use repository-relative Markdown links for citations and normal relative links
between wiki pages. Distinguish facts, interpretations, historical claims, and
open questions. Preserve negative findings and contradictions.

## Operations

### Ingest

When asked to ingest or refresh project knowledge:

1. Register new sources in `wiki/raw/index.md`.
2. Read the source and identify which existing concepts it changes.
3. Update or create the smallest useful set of compiled pages.
4. Strengthen cross-links and source citations.
5. Update `wiki/index.md`.
6. Append one parseable entry to `wiki/log.md`:
   `## [YYYY-MM-DD] ingest | short title`.
7. Run `make wiki-lint`.

### Query

Start from `wiki/index.md`, synthesize from the relevant pages, and cite the
underlying source files. If the answer creates durable new knowledge, offer or
perform a wiki update when the user's request authorizes edits.

### Lint

`make wiki-lint` checks frontmatter, the global index, internal links, source
citations, orphans, and log syntax. Fix errors before considering a wiki update
complete. Warnings require review but do not always require a change.

## Repository-specific guardrails

- Treat `config/playbook*.yaml`, `scripts/playbooks/refit_*.yaml`, tracked
  research baselines, and historical conclusions in `docs/META_AGENT.md` as
  frozen unless the user explicitly authorizes a new promotion or revision.
- Preserve temporal alignment and no-lookahead guarantees in backtests.
- Any claim of strategy effectiveness must include its validation window,
  baseline, statistical evidence, and known leakage or selection risks.
- The system is a research and monitoring tool, not an automated trading
  system and not investment advice.

---
> Source: [Linjay/quant-lab](https://github.com/Linjay/quant-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
