## trip-packer

> This file sets the ground rules for how Claude works in this codebase.

# CLAUDE.md

This file sets the ground rules for how Claude works in this codebase.
Read it at the start of every session. These rules apply to all tasks.

---

## Machine & environment context

- This project is developed on a **work laptop** — be conservative with any system-level actions
- Do not install packages, run scripts, or modify system config without explicit confirmation
- Assume corporate security tools may be monitoring file changes and network activity

---

## File access

- Only read and write files **within this project folder**. Never touch anything outside it.
- Before editing any file, show what you plan to change and wait for confirmation. No silent edits.
- Never delete files without asking first. Deletions are irreversible outside git.

---

## Planning & transparency

- For any non-trivial action, **explain what you're about to do, why, and what the risks are** before doing it.
- Ask clarifying questions rather than filling gaps with assumptions.
- Be explicit about uncertainty — distinguish between **Verified**, **Assumed**, and **Estimated**. Never present a guess as a fact.
- If something wasn't run or tested, say so. Don't claim it works.

---

## Change management

- Make **one change at a time**. Confirm it works before moving on.
- Prefer **incremental, reversible changes** over rewrites.
- Ask before: modifying many files at once, refactoring existing code, changing auth or config, modifying a database schema, or replacing working functionality.
- Prefer simple, maintainable solutions. Avoid premature optimisation and unnecessary complexity.

---

## Security & secrets

- **Never hardcode API keys, tokens, passwords, or credentials** anywhere in the code.
- Always use environment variables for secrets. Reference them by name only — never log or expose their values.
- Flag any change touching authentication, API keys, or tokens before making it.
- If you spot a secret that may have been accidentally committed, flag it immediately.

---

## Dependencies

- **Do not install packages without asking first.** On a work laptop this can trigger security alerts.
- When proposing a new dependency, explain: why it's needed, what alternatives exist, and the tradeoffs. Keep the dependency set lean.
- Prefer well-maintained, widely-used packages over niche or unmaintained ones.

---

## Git

- **Never run `git push` without being asked.** Nothing leaves the local machine automatically.
- Use branches for any significant work. Name them clearly with a prefix: `feat/`, `fix/`, or `chore/`. The branch name should be self-explanatory from a list.
- Never merge to `main` without explicit permission.
- Write clear, descriptive commit messages — one logical change per commit.

---

## Tech stack

- Stack is **TBD** — when recommending technologies, explain the reasoning and flag trade-offs.
- Once a stack decision is made, record it in `PRD.md` or `BUILDPLAN.md` before writing code against it.
- Don't introduce frameworks or patterns not agreed on — flag suggestions separately.

---

## UI & visual design

Before writing any UI, read DESIGN.md. Use the colour tokens, typography scale,
spacing values, and component patterns defined there throughout. Do not default
to generic styles — follow that direction for every element you build.

---

## Task completion summary

At the end of every significant task, provide a summary covering:

1. **What changed** — files modified and why
2. **Test results** — what was run and what passed or failed
3. **Known risks** — anything that could break or needs watching
4. **What couldn't be verified** — anything untested or assumed
5. **Recommended next steps**
6. **Confidence score** — 1–10, with a brief reason

---

## Key project files

| File | Purpose |
|---|---|
| `CLAUDE.md` | Ground rules for this codebase (this file) |
| `PRD.md` | What we're building — product requirements |
| `BUILDPLAN.md` | How and in what order we're building it |
| `DESIGN.md` | Visual direction — colours, typography, spacing, components |
| `BACKLOG.md` | Ideas and changes logged for later — not in scope yet |

---

*Last updated: project start · Update this file as the project evolves.*

---
> Source: [norwsn47/trip-packer](https://github.com/norwsn47/trip-packer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
