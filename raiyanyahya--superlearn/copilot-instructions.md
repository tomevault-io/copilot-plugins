## superlearn

> Superlearn turns "I want to learn X" into a researched, interactive learning

# Superlearn

Superlearn turns "I want to learn X" into a researched, interactive learning
board served at http://localhost:4321. It ships as a Claude Code plugin and as
a standard Agent Skill for OpenAI Codex, Kilo (formerly Kilo Code), and any
agent that reads `SKILL.md`.

## If the user asks to learn, study, or research a topic

Follow the pipeline in `.agents/skills/superlearn/SKILL.md` — research first,
plan, iterate until saturated, author and validate the board, serve, keep
iterating. Do not improvise a different flow, and never author a board before
research is saturated. (`skills/superlearn/SKILL.md` is the same playbook in
its Claude Code plugin edition.)

## Working on this repo

- Zero dependencies by design: every script in `scripts/` is Python 3.9+
  standard library, and the app is one file (`app/index.html`) with no build
  step. Keep it that way.
- Run the tests with `python3 -m unittest discover -s tests -v`.
- The board contract lives in `scripts/validate_board.py`; if you change it,
  update the schema documentation in `skills/superlearn/SKILL.md` (and vice
  versa) so they never disagree.
- `.agents/skills/superlearn/SKILL.md` must end with the body of
  `skills/superlearn/SKILL.md` verbatim — a unit test enforces this. Edit the
  Claude edition first, then regenerate or mirror the portable copy.
- The scrapers must stay polite: identify honestly, tiny request volume, no
  login walls, no paywall or CAPTCHA circumvention — see "The researched data
  isn't ours" in the README.

---
> Source: [raiyanyahya/Superlearn](https://github.com/raiyanyahya/Superlearn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
