## idea-to-product-skill

> >


# Idea-to-Product — Master Skill

A structured, stage-gated workflow that guides an idea from first spark to
tested, launched digital product. Each phase ends with a **Gate** — a
decision point requiring explicit sign-off before the next phase begins.

---

## Agent Compatibility

This skill is designed to run on any agent environment:

| Environment     | Notes                                              |
|-----------------|----------------------------------------------------|
| Claude Code     | Full support. Use subagents for parallel research. |
| Gemini CLI      | Full support. Run phases sequentially.             |
| Claude.ai       | Full support. No subagents — run phases inline.    |

---

## Skill Map

Each phase has its own reference file. Read only the file for the active phase.

| Phase | File | Sub-skills used |
|-------|------|-----------------|
| 1 — Brainstorming | `references/phase-1-brainstorm.md` | `brainstorming`, `writing-plans` |
| 2 — Market Validation | `references/phase-2-market.md` | `marketing-psychology`, `content-strategy`, `seo-audit`, `marketing-ideas` |
| 3 — Funnel & Product Decision | `references/phase-3-funnel.md` | `launch-strategy`, `page-cro`, `form-cro`, `email-sequence`, `marketing-ideas` |
| 4 — Content & Copy | `references/phase-4-content.md` | `copywriting`, `product-marketing-context`, `doc-coauthoring`, `docx`, `pptx` |
| 5 — Build & Test | `references/phase-5-build.md` | `frontend-design`, `webapp-testing`, `analytics-tracking`, `form-cro`, `seo-geo`, `executing-plans` |

---

## Core Principles

**One phase at a time.** Never jump ahead. Each Gate must be explicitly
approved by the user before loading the next phase file.

**One question at a time.** When gathering information, ask questions
individually. Never list multiple questions in one message.

**Gate = hard stop.** At the end of every phase, present a structured Gate
summary and ask: *"Weiter zu Phase X?"* — do not proceed without a clear yes.

**Language follows the user.** If the user writes in German, respond in German.
If they switch languages, follow.

**Context is king.** At the start of each new phase, briefly recap what was
decided in the previous phase. Never make the user repeat themselves.

---

## How to Start

When this skill is triggered, execute these steps in order:

### Step 0 — skills.sh Live-Check (always first)

Before anything else, fetch https://skills.sh and scan for skills relevant
to this workflow. Compare against the Sub-skill Map above.

```
GET https://skills.sh
```

Look specifically for new or updated skills in these categories:
- Brainstorming / ideation / planning
- Market research / validation / competitive analysis
- Marketing / copywriting / CRO / email / SEO
- Product / course / funnel building
- Testing / analytics / launch

For each newly discovered relevant skill, check if it improves or replaces
an existing entry in the Sub-skill Map.

**Report findings to the user** in this format before proceeding:

```
## 🔍 skills.sh Check — [Datum]

**Bekannte Skills bestätigt:** [N von N verfügbar]

**Neu gefundene relevante Skills:**
- `[skill-name]` ([owner/repo]) → passt zu Phase [N]: [1 Satz warum]

**Empfehlung:** [Soll ein neuer Skill installiert / genutzt werden? Ja/Nein + Begründung]

➡️ Skill-Map für diese Session aktualisiert.
```

If no new relevant skills are found, state that briefly and continue.
If the fetch fails (no internet / tool unavailable), log the failure and
continue with the existing Sub-skill Map — never block the workflow.

---

### Step 1 — Greet and orient

Briefly explain the 5-phase journey in 2–3 sentences.

### Step 2 — Load Phase 1

Read `references/phase-1-brainstorm.md` and begin.

---

If the user has already done some phases ("I already have a market analysis"),
still run Step 0 (skills.sh check), then ask which phase to start at and
load that reference file directly.

---

## Gate Protocol

Use this exact structure at the end of every phase:

```
## ✅ Gate [N] — [Phase Name] abgeschlossen

**Kernerkenntnisse:**
- [bullet 1]
- [bullet 2]
- [bullet 3]

**Entscheidung:** [Go / No-Go / Überarbeiten]

**Nächste Phase:** [Name der nächsten Phase]

➡️ Bereit für Phase [N+1]?
```

Wait for an explicit "ja", "weiter", "go" or equivalent before proceeding.
If the user says "überarbeiten" (revise), go back within the current phase.
If the user says "stopp" or "nein", close the workflow gracefully and summarize
what was completed.

---

## Sub-skill Integration

When a sub-skill is listed for a phase, follow this pattern:

1. **Check availability** — look for the sub-skill in the agent's skill list.
   If available, load and follow it. If not, apply its principles from memory
   (each reference file contains a condensed version of the key sub-skill
   principles as a fallback).

2. **Cross-reference with skills.sh results** — if the Step 0 check found a
   new skill that fits this phase better than the listed one, prefer the newer
   skill and note the substitution to the user.

3. **Never block on missing sub-skills.** The workflow continues regardless.
   Sub-skills enhance quality; they don't gate progress.

---

## Output Files

Throughout the workflow, maintain a living project document. Save it as:

```
docs/idea-to-product/[YYYY-MM-DD]-[idea-slug].md
```

Each phase appends its results to this document. The file becomes the single
source of truth for approvals, decisions, and content drafts.

If a filesystem is not available (e.g., Claude.ai chat), maintain the document
as a structured conversation artifact and offer to export at the end.

---

## Quick Reference — Phase Sequence

```
START
  │
  ▼
Phase 1: Brainstorming ──────── Gate 1 ─── No-Go → STOP
  │
  ▼
Phase 2: Marktanalyse ───────── Gate 2 ─── No-Go → STOP
  │
  ▼
Phase 3: Funnel & Produkt ───── Gate 3 ─── Überarbeiten → back to Phase 2/3
  │
  ▼
Phase 4: Inhalte & Texte ────── Gate 4 ─── Auftraggeber-Freigabe erforderlich
  │
  ▼
Phase 5: Erstellung & Test ──── Gate 5 ─── Launch-Freigabe
  │
  ▼
LAUNCH
```

---

## Now: Load Phase 1

Read `references/phase-1-brainstorm.md` and begin.

---
> Source: [r0bert011/idea-to-product-skill](https://github.com/r0bert011/idea-to-product-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
