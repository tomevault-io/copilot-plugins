## skill-vet

> > **First read `HANDOFF.md`** for the current project state and `.claude/memory/`

# CLAUDE.md — instructions for Claude Code sessions in this repo

> **First read `HANDOFF.md`** for the current project state and `.claude/memory/`
> for accumulated context. This file is short by design — it just tells you
> the *conventions* for working in this repo.

## Memory system

Two layers:

### 1. Rich in-flight memory (your job — the ceiling)

Write to `.claude/memory/*.md` whenever a real project decision or artifact
lands. Use the same file format as the auto-memory system in the user's
global CLAUDE.md — YAML frontmatter (`name`, `description`, `metadata.type`)
+ body linked from `MEMORY.md`.

**Trigger events that MUST produce an in-flight memory update:**

- A verdict-policy change (e.g. adding a rule to `STYLE_RULES`)
- A new eval methodology (adjudicator, style-tuning, weighting change)
- An upstream issue filed or PR opened
- A discovered false-positive or false-negative pattern worth remembering
- Any decision the user paused to explicitly discuss and resolve

**Trigger events that MAY produce one, at your discretion:**

- A milestone commit (repo-wide refactor, corpus refresh)
- A blocker hit + resolved (e.g. Bedrock throttling → serialization lock)
- A user preference confirmed multiple times

**Trigger events that MUST NOT produce one:**

- Routine tool calls (Read, Grep, single-file Edit)
- Failed experiments you rolled back
- Cosmetic changes

Keep entries short — 1-3 paragraphs, plus a **Why:** line and a **How to
apply:** line. Follow the template in the existing memory files.

### 2. Stop-hook session log (safety net — the floor)

`.claude/hooks/stop-summarize.sh` runs on session end and appends a
mechanical summary to `.claude/memory/session_log.md`. This is *only* the
list of user messages + tool-use count — no rich context. If your in-flight
writes cover the important stuff, this log becomes almost redundant. If
you forgot, at least the log exists.

**Don't rely on this as your primary mechanism.** It's the fallback for the
case where you (Claude) got distracted and never wrote a proper memory
entry. Every session should produce at least one hand-written memory update
for a non-trivial project change.

## Repo-scoped conventions

- **Verdict policy** and `STYLE_RULES` live in `eval/policy.py`. If you
  change either, update `.claude/memory/verdict_policy.md` **and** run
  the eval to see the impact before committing.
- **Adjudicator** is on by default (see `.claude/memory/adjudicator.md`).
  Don't remove the module-level `threading.Lock`; it's load-bearing.
- **Costs are real.** Confirm before kicking off any full corpus eval
  (~$50 Bedrock, ~1h wall). Small runs / smoke tests are fine unattended.
- **Two remotes**: `origin` = internal Smartsheet GitLab, `github` = public
  `rcha0s/skill-vet` on GitHub. Auth for both is in the gitignored `.env`.
  Push to both when a change is durable; smoke work stays local.

## Escalation for cross-cutting decisions

Anything that would change the shape of the verdict policy, the wrapper's
CLI contract, or the upstream Cisco pitch — pause and check with the user
before implementing. Memory-worthy decisions belong to the user, not the
agent.

---
> Source: [rcha0s/skill-vet](https://github.com/rcha0s/skill-vet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
