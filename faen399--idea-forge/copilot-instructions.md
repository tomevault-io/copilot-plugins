## idea-forge

> You are working in the **Idea Forge** repo. This is an idea inbox where the user drops ideas and agents build from them.

# Idea Forge — Agent Instructions

You are working in the **Idea Forge** repo. This is an idea inbox where the user drops ideas and agents build from them.

## Your workflow

1. **Read ideas/** — Look for files with `status: new` or check what the user points you to
2. **Understand the idea** — Read it fully. If anything is unclear, ask before building.
3. **Plan first** — Create a brief plan in `output/<idea-name>/plan.md` before writing code
4. **Build** — Write output to `output/<idea-name>/`. This can be code, docs, designs, whatever the idea calls for.
5. **Update status** — Change the idea's `status:` from `new` to `in-progress` or `done`
6. **Commit & push** — Commit your work with a clear message referencing the idea

## Rules

- One idea = one output directory
- Don't modify the idea file beyond updating `status:`
- If an idea is too vague, ask the user — don't guess
- Prefer simple, working implementations over over-engineered ones
- Leave a `output/<idea-name>/README.md` explaining what you built and how to use it

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FAeN399) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
