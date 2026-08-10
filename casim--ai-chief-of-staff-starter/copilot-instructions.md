## ai-chief-of-staff-starter

> Conventions for any AI/coding assistant working in this repo. This file is

# AGENTS.md

Conventions for any AI/coding assistant working in this repo. This file is
tool-agnostic — it should make sense to whatever agent you use (ChatGPT, Claude,
Gemini, Cursor, Notion AI, etc.).

## What this repo is
A personal "vault" of markdown notes that acts as an agent's **long-term
memory**, plus a set of **skills** the agent
runs on the user's behalf. The vault is the source of truth; skills describe
*intent*, not code.

## Folder map
- `daily/` — daily notes, one file per day named `daily/YYYY-MM-DD.md`. Contain
  tasks (checkbox lists), free-text notes, and a `## Focus` section. These are
  the richest signal about what the user is doing — read them first.
- `meetings/` — 1:1 and meeting notes (often synced from an external tool). One
  running file per person, or one file per one-off meeting.
- `concepts/` — the user's evolving views on topics. **Every concept note starts
  with a `## Summary`** describing the current view.
- `meta/` — context that helps you understand the user: `goals.md` (yearly /
  quarterly goals) and `topics.md` (what's signal vs noise).
- `working/` — scratch space for analysis "notebook" projects (code + deps +
  README). Git-ignored by default.
- `.cursor/skills/` and `.claude/skills/` — one folder per skill, each
  containing a `SKILL.md` with YAML frontmatter (`name`, `description`) followed
  by markdown steps. `_template/` is the blank skeleton for new skills. The two
  folders hold identical copies so the project auto-loads skills in both Cursor
  (`.cursor/skills/`) and Claude Code (`.claude/skills/`).

## Key conventions
- Daily notes: `daily/YYYY-MM-DD.md`.
- Concept notes: must have a `## Summary` at the top.
- Meeting notes: live in `meetings/`.
- Scratch / analysis work: lives in `working/`, in its own subfolder with a
  README and a dependency file.
- Skills: live in `.cursor/skills/<skill-name>/SKILL.md` and
  `.claude/skills/<skill-name>/SKILL.md` (identical copies). They are
  instructions in plain English, not programs.

## How to behave
- Treat the vault as memory: read relevant notes before acting.
- Prefer appending/updating over overwriting the user's notes; never silently
  destroy local edits.
- Anything needing an external tool (calendar, chat, a notes provider) is
  "connect your own tool" — don't assume a specific vendor.
- Secrets belong in a secrets manager / vault (OS keychain, password manager,
  or cloud secrets manager), never in tracked files. Use a git-ignored `.env`
  only as a local fallback.

---
> Source: [casim/ai-chief-of-staff-starter](https://github.com/casim/ai-chief-of-staff-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
