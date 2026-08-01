## evc

> Marketing agent workspace powered by EVC.

# {{COMPANY_NAME}} — Agent Instructions

Marketing agent workspace powered by EVC.

---

## Operating Principles

Read and follow `SOUL.md` — 7 principles that govern every action.

## Brand Context

- **Brand rules:** `brand/canon-rules.json` — banned phrases, required language, USPs, compliance rules
- **Content config:** `content-engine/config.md` — audience segments, posting schedule, tone of voice
- **Logos and templates:** `brand/logos/` and `brand/templates/`

Load `brand/canon-rules.json` before writing any external-facing content.

## Context Modes

Load the appropriate mode before starting work:

| Mode | File | When |
|------|------|------|
| Content Mode | `.claude/contexts/content-mode.md` | Writing social posts, blog drafts, any copy |
| Build Mode | `.claude/contexts/build-mode.md` | Creating visual assets, HTML templates, exports |
| Research Mode | `.claude/contexts/research-mode.md` | Market research, competitor analysis, deep dives |

## Key Rules

- **Copy first, assets second** — never design visuals for unapproved copy
- **Staging before production** — preview in `staging/` before CMS push
- **No auto-posting** — always draft, never auto-send to external channels
- **Every claim sourced** — trace to canon or flag as unverified
- **Run stop-slop on everything** — no AI writing patterns in outbound content

## Working Style

- When you have enough information to act, act. Don't re-derive established facts or re-litigate decisions already made.
- Pause for the user only when the work genuinely requires them: a destructive or irreversible action, a real scope change, or input only they can provide. The human gates defined in pipelines always apply.
- Before reporting progress or a completed checklist, audit each claim against a tool result from this session — never report a step done that you didn't verify.

## Memory

The system learns automatically:
- `content-engine/memory/lessons.md` — accumulated lessons from QA failures and production
- `content-engine/memory/learning-log.md` — content performance patterns
- Read these before starting content work — they contain hard-won knowledge

## Workspace

- **Output:** `content-engine/output/` — weekly social content, blog articles, reports
- **Exports:** `content-engine/exports/` — final PNGs/PDFs ready to upload
- **Staging:** `staging/` — HTML previews before CMS push

## Commands

Use `/project:help` to see all available commands grouped by use case.
Use `/project:monday` to start the week.
Use `/project:friday` to close the week.

---
> Source: [SaigonXIII/evc](https://github.com/SaigonXIII/evc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
