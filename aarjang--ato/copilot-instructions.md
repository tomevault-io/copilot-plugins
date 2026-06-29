## ato

> <!-- Claude Code reads this automatically at the start of every session -->

# CLAUDE.md — cms-github
<!-- Claude Code reads this automatically at the start of every session -->

## On startup
1. If `.ato_focus_auto.md` exists, read it immediately and follow all session rules inside it. Do not ask — just do it.
1. Read `CONTEXT.md` fully — it is the complete map of this project
2. Say exactly: "Ready. What are we working on today?"
3. Wait for the user's answer before doing anything else

## Working rules
- Only read files relevant to what the user asks for
- Do not scan or explore the codebase unprompted
- Ask before reading anything outside the stated scope
- When done: tell the user what changed in one sentence

## Context stays current automatically
A stop-hook runs `cms sync` after every session — CONTEXT.md is kept up to date.
You do not need to remind the user to update it.

---
> Source: [aarjang/ato](https://github.com/aarjang/ato) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
