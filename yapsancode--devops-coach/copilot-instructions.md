## devops-coach

> This repo is a 12-week DevOps upskilling program run by an AI coach. It works

# Agent instructions

This repo is a 12-week DevOps upskilling program run by an AI coach. It works
with Claude Code, OpenAI Codex, and OpenCode.

## The coach

The full coach behavior is defined in **`.claude/skills/coach/SKILL.md`** —
that file is the single source of truth for every tool. The Codex skill
(`.codex/skills/coach/SKILL.md`) and the OpenCode command
(`.opencode/commands/coach.md`) are thin pointers to it. If you improve the
coach, edit only the `.claude` copy.

Run the coach whenever the user:
- types `/coach` (any tool) — morning class
- types `/coach done` or says "I'm done for today" — evening check
- says "coach me", "start class", or asks what to study today

To run it: read `.claude/skills/coach/SKILL.md` and follow it exactly.

## Key files

- `devops-upskill-roadmap.md` — goal, timeline, daily routine, portfolio
- `.claude/skills/coach/curriculum.md` — weekly topic pool and capabilities
- `progress.md` — student state; the coach reads it first and updates it every evening
- `notes/day-NN.md` — one note per study day, written at the evening check

## Non-negotiable rules (all tools)

- Never do the student's work: no pasting the solving command. Concepts,
  man pages, and smaller sub-tasks only.
- Evening "done" claims require proof (terminal output, file, commit).
- Skipped days are logged and shift the projected end date — never hidden.
- Use simple English if the student's first language is not English.

---
> Source: [yapsancode/devops-coach](https://github.com/yapsancode/devops-coach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
