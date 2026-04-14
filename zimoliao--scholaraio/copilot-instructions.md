## scholaraio

> This project is an AI-powered research terminal. Full project instructions are in `AGENTS.md` at the repository root. **Read `AGENTS.md` before proceeding with any task.**

# ScholarAIO — Cursor Rules

This project is an AI-powered research terminal. Full project instructions are in `AGENTS.md` at the repository root. **Read `AGENTS.md` before proceeding with any task.**

## Quick Reference

- CLI entry point: `scholaraio --help`
- Python package: `scholaraio/`
- Paper data: `data/papers/<Author-Year-Title>/` (meta.json + paper.md)
- User output: always write to `workspace/`, never to project root or `scholaraio/`
- Skills (reusable workflows): `.claude/skills/*/SKILL.md`
- Tests: `python -m pytest tests/ -v`
- Code style: Google-style docstrings for library modules, Chinese for CLI output, English for code comments
- Config: `config.yaml` (git-tracked), `config.local.yaml` (secrets, not tracked)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZimoLiao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
