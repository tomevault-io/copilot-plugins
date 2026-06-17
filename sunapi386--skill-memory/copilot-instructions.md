## skill-memory

> Use when starting any task to recall lessons from past runs, and after finishing any task to record lessons learned — gives your agent compounding memory across sessions


# Skill Memory

Agents are stateless. Every session starts from zero. This skill gives your agent a persistent, append-only lesson log so knowledge compounds across runs.

The rule: **read before you act, write after you learn.**

## How it works

A tiny CLI (`skill-memory`) writes one JSON line per lesson to `~/.skill-memory/lessons.jsonl`. No server, no database, no config. The file is yours — grep it, version it, share it, delete it.

## Commands

```bash
skill-memory recall <keywords>   # search past lessons (no args → last 20)
skill-memory learn "<lesson>"    # append a new lesson
skill-memory list                # dump everything
skill-memory count               # how many lessons you've logged
```

## Protocol for agents

**Step 1 — before any non-trivial task, recall.**

```bash
skill-memory recall <keywords from the task>
```

If you get hits, read them. Treat them as context the user would have told you if they remembered.

**Step 2 — after any task, learn if anything surprised you.**

```bash
skill-memory learn "When <situation>, <do X>. Because <reason>."
```

Structure lessons as `situation → action → reason`. The reason is the load-bearing part — it lets future-you judge edge cases instead of blindly following the rule.

## When to log

- A test failure that took more than one attempt to fix
- A library quirk or footgun that surprised you
- A user correction ("stop doing X", "always do Y")
- A pattern that worked and should repeat
- A dead end — so future runs don't repeat the waste

## When NOT to log

- Code that's already in the file (the file is authoritative)
- One-off trivia with no reuse value
- Anything that will rot in a week (ephemeral task state)
- Things already documented in CLAUDE.md / README / existing skills

## When to recall

- Before touching an unfamiliar codebase or library
- Before picking a tool where you have a choice
- When something feels familiar but you can't place it
- When the user gives a terse instruction — past lessons often have the missing context

## The compounding part

One lesson is noise. Fifty lessons is a playbook. Five hundred lessons from a community is a second brain.

Lessons are plain JSONL — share them, merge them, publish them. That's the self-improvement loop: each agent's mistakes become the next agent's shortcuts.

---
> Source: [sunapi386/skill-memory](https://github.com/sunapi386/skill-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
