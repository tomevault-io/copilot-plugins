## itr-wala

> Agent skill for filing Indian income tax returns (ITR), FY 2025-26 / AY 2026-27.

# itr-wala

Agent skill for filing Indian income tax returns (ITR), FY 2025-26 / AY 2026-27.

- The skill lives at `skills/itr-wala/SKILL.md` (Agent Skills standard -
  works in Claude Code, Codex CLI, and Gemini CLI). Codex discovers it
  automatically inside this repo via `.agents/skills`.
- All tax arithmetic is done by `skills/itr-wala/scripts/tax_engine.py`
  (stdlib-only Python, golden-tested). Agents must never compute tax figures
  themselves - see the Iron Rules in SKILL.md.
- Run the test suite before changing the engine:
  `python3 skills/itr-wala/scripts/test_tax_engine.py`
- Rates are pinned to AY 2026-27. A new assessment year means updating the
  constants block in `tax_engine.py`, the reference docs, and the golden
  tests together.

---
> Source: [karanb192/itr-wala](https://github.com/karanb192/itr-wala) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
