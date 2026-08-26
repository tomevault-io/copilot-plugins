## source-words

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Source Words is the authoritative shared baseline for a plain-text development-context system. It defines decisions, backlog, discussions, ideas, retros, and the lifecycle rules that connect them. The baseline remains usable as copied Markdown; the repository also contains an optional guarded updater and its standard-library tests.

## Repository Structure

Default structure when installed as a complete system:

```
.source-words/
├── backlog/       # Work to be done (features, bugs, refactoring)
├── decisions/     # Immutable records of choices made (ADR-style)
├── discussions/   # Living documents for ongoing thinking
├── ideas/         # Undecided possibilities that may never become work
└── retros/        # Durable context extracted from implementations
```

Complete consumer installations normally live at `.source-words/`. Individual subsystems can also be installed standalone or at a custom location.

Backlog, decisions, and discussions keep shared guides in `core/`. Ideas and retros use a top-level `README.md`.

If `.source-words/LOCAL.md` exists in a consumer, read it after the canonical guides. Project-specific guidance belongs there; do not edit canonical baseline files for local conventions.

## Working with Each System

### Backlog

- **File naming:** `YYMMDD-short-description.md`
- **Location:** Story files go in the backlog root (not in `core/`)
- **Path flexibility:** Can be at `.source-words/backlog/` (full install), `backlog/` (standalone), or a custom location
- **Stories describe problems**, not solutions - include code examples with file paths
- **Review process:** Read `{backlog}/core/REVIEW-PROCESS.md` to execute the full review workflow (uses `{BACKLOG_ROOT}` placeholder for paths)
- **Checkpoint:** `{backlog}/.last-review` tracks when backlog was last reviewed

### Decisions

- **File naming:** `NNNN-short-description.md` (four-digit sequential number)
- **Location:** Files go in `decisions/` root (not in `core/`)
- **Accepted decision content is immutable** - create a new decision when the choice changes
- **Status lifecycle:** Proposed → Accepted → (Deprecated | Superseded); lifecycle status and supersession links may be updated
- **Template fields:** Status, Date, Scope (Architecture/UX/Product-Direction), Tags

### Discussions

- **File naming:** `YYMMDD-topic-name.md` (date prefix shows when discussion started)
- **Location:** Files go in `discussions/` root (not in `core/`)
- **Points-based structure:** Each topic/alternative gets its own H2 section with all related reasoning grouped together
- **Resolved discussions** are archived when still valuable and otherwise deleted

### Ideas

- **File naming:** `YYMMDD-short-name.md`
- **Lifecycle:** Optional and disposable; promote to a discussion or backlog story when warranted
- **Structure:** Free-form, with no review obligation

### Retros

- **File naming:** `YYMMDD-short-description.md`, or a matching folder when multiple artifacts are needed
- **Lifecycle:** Durable implementation context, normally committed with the implementation
- **Structure:** Free-form after the implementation reference and date

### Scratch

- Temporary and always safe to delete
- Never reference scratch files from elsewhere in the repository

## Image Assets

For any system that needs images:
- Store in `{system}/assets/{filename-without-extension}/`
- Reference from a record in the system root: `![desc](assets/record-name/image.png)`
- Only image formats supported (PNG, JPG, GIF, SVG, WebP)

## Key Principles

- Plain text in git - no databases, no special tools
- Stories/discussions describe *what* and *why*, not step-by-step *how*
- Decisions capture rationale and consequences, not just the choice
- Durable decisions do not link to disposable backlog, idea, or scratch artifacts
- Temporary artifacts may link to durable decisions, not the reverse
- ADRs preserve durable context, the decision, its rationale, and lasting consequences; implementation detail stays in code, tests, architecture docs, runbooks, or plans

## Distribution Tooling

`bin/source-words-baseline` checks, diffs, installs, and updates revisioned consumer baselines. Its allowlist is `distribution/baseline-files.txt`; never broaden that list to consumer-owned stories, decisions, discussions, ideas, retros, scratch, checkpoints, assets, runbooks, or local guidance.

Run its tests with:

```bash
python3 -m unittest discover -s tests -p 'test_*.py'
```

---
> Source: [ggratte/source-words](https://github.com/ggratte/source-words) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
