## remmd

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is remmd?

A document platform where **links** (bilateral agreements) between document sections create a trust network. Content edits are immediate and versioned. Links are the only reviewed entity — thread-based, like PRs. When linked content changes, the graph ripples and counterparties review.

**Core loop:** Author creates content (auto `@refs`) → proposes links between sections across documents → content changes trigger graph walk → author reaffirms/withdraws → counterparty reviews in thread → link aligned or iterate.

## Domain Model

Seven processes, four aggregates + adapter boundary:

**Aggregates:** Document (sections with auto `@refs`, versions, tags, source), Link (section(s)→section(s), relationship type, rationale, thread, intervention per endpoint, derived state), Policy (selector, predicate, surface, severity, violations, waivers), Subscription (tag expression, fire events).

**Boundary:** Source Adapter (git, Jira, Confluence, Figma — `emit_sections`, `on_change`, `render_preview`).

**Link states (derived):** `pending` → `aligned` (both approved) → `stale` (content changed) → iterate in thread → `aligned`. Also: `broken` (section deleted/unresolvable), `archived` (human closed).

**Intervention levels (per endpoint):** `watch` (dashboard only) → `notify` (normal task) → `urgent` (high priority + notification) → `blocking` (blocks gates until resolved).

**Key invariants:**
- Content edits are immediate and versioned — no draft/activate for content
- Links are the only reviewed entity — bilateral approval required
- Trust attaches to exact section snapshots
- Changed side must reaffirm or withdraw impacted links
- Review is thread-based iteration (like PR code review)
- AI drafts, humans approve — service principals never satisfy trust actions
- Graph walks on every change — blast radius shown before action
- Stale context cannot approve current truth

## UX Architecture

Canvas-oriented, three zoom levels:
- **Close (Author mode):** Full editor, link panel sidebar, section border state indicators
- **Medium (Document mode):** Sections as blocks, links as lines, resolve stale items
- **Far (Network mode):** Force-directed graph, nodes=documents, edges=links

Design philosophy: sparse by default, progressive disclosure, no empty state onboarding. Grayscale Editorial style.

## Spec Files

- `spec/remmd.md` — Normative spec (source of truth)
- `spec/event-storming.md` — Event storming aligned to remmd.md
- `spec/gaps.md` — UX/interaction gaps identified but not yet designed
- `spec/ux-spec.md` — UX specification (needs reconciliation with remmd.md)

## Tech Stack

- **Language:** Go
- **Database:** Embedded SQLite (modernc.org/sqlite, pure Go, no CGO)
- **CLI framework:** Cobra
- **Architecture:** Domain-driven, event-sourced, repository pattern

## Architecture

This project uses C3 docs in `.c3/`.
For architecture questions, changes, audits, file context → `/c3`.
Operations: query, audit, change, ref, rule, sweep.
File lookup: `c3x lookup <file-or-glob>` maps files/directories to components + refs.

## Build & Dev Commands

_To be populated once project is scaffolded._

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lagz0ne) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
