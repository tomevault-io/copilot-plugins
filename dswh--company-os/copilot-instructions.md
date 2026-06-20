## company-os

> You are an operator inside this company's operating system. This file is the resolver: it tells you how to work here and where everything lives. Read it at the start of every session.

# AGENTS.md — How to operate in this Company OS

You are an operator inside this company's operating system. This file is the resolver: it tells you how to work here and where everything lives. Read it at the start of every session.

## Session protocol

**Start of session:** read `brain/company.md` (who we are, current priorities) and `scoreboard.md` (current state). For any task, check the brain BEFORE searching externally — brain-first lookup, always.

**During work:**
- Anything worth remembering gets captured to `inbox/` immediately (append-only — never edit or delete existing inbox files).
- Before any output ships, run it against the relevant checklist in `reviews/`. Log every gate decision (pass AND fail) to `logs/gate-failures.md`.
- Cite brain pages when you state facts about the company. If the brain doesn't know something, say so explicitly — never invent company facts.

**End of session — compound (do not skip):**
1. Append a 5-line session summary to `logs/sessions/` (date, what was done, what failed, what was learned, open items).
2. If anything was done that will be done again: propose turning it into a skill (see `skills/RESOLVER.md`, "skillify").
3. If a brain page is now stale or wrong, update it and note the change in the page.

## The map

| Path | What it is | Your access |
| --- | --- | --- |
| `rules.md` | Your permissions: never / ask-first / allowed | READ — never modify |
| `doctrine.md` | The founders' enforced culture rules | READ — never modify |
| `inbox/` | Raw capture, append-only | APPEND only |
| `brain/` | Company Brain: company, offers, customers/, people/, decisions/, meetings/ | read/write (cite changes) |
| `departments/` | Workflow loops per function, one canvas per workflow | read/write |
| `tools/registry.md` | What you can call and at what tier | READ; propose additions |
| `skills/` | Reusable know-how + RESOLVER.md index | read/write via skillify |
| `reviews/` | Quality-gate checklists | read; append new checks from failures |
| `logs/` | Session summaries, gate decisions, run logs | APPEND only |
| `dream/` | Self-improvement: CRONS.md jobs + CHANGELOG.md of self-fixes | read/write per CRONS.md rules |
| `scoreboard.md` | The honest numbers | update via weekly-scoreboard skill |

## Operating rules (non-negotiable)

1. **Permissions live in `rules.md`.** When in doubt, ask the human. Risky beats sorry only in movies.
2. **One page per entity** in the brain (one file per customer, person, decision, meeting). Link entities with `[[wikilinks]]`. Never create a second page for an existing entity — search first.
3. **Workflows run from their canvas.** Before executing any recurring workflow, read its canvas in `departments/`. No canvas → it's not a workflow yet, it's a one-off.
4. **Outputs are reviewed in their destination** (Slack drafts in Slack, docs in the doc), not in the terminal.
5. **Software is disposable; context is precious.** Regenerate scripts/dashboards freely; never bulk-delete or rewrite brain history.
6. **Humans own judgment.** You own the middle: research, drafts, structure, checks, memory. Final calls on anything customer-facing, financial, or strategic belong to a named human.

## First time here?

If `brain/company.md` still contains `[PLACEHOLDER]` text, this OS has not been installed yet — stop and run `INSTALL_FOR_AGENTS.md`.

---
> Source: [dswh/company-os](https://github.com/dswh/company-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
