## project-x

> - Read `docs/DEVLOG.md` at session start (it contains the "Active Focus" section and recent entries)

# Crystal — Cursor Rules

## Session workflow
- Read `docs/DEVLOG.md` at session start (it contains the "Active Focus" section and recent entries)
- Write working notes to `docs/SESSION.md` during the session
- Before each commit: append a dated entry to `docs/DEVLOG.md` (what changed, decisions)
- Update the "Active Focus" section in `DEVLOG.md` if priorities shifted
- Clear `docs/SESSION.md` at session end

## DEVLOG hygiene
- Keep only the most recent ~5 entries in `DEVLOG.md`
- When adding a new entry would exceed ~5, move the oldest entry to `docs/DEVLOG_ARCHIVE.md`
- Entry format: "What changed" (concise bullets) + "Decisions" (only non-obvious ones). Drop sections with nothing to say.
- Do not duplicate known-issue descriptions across entries — the "Active Focus" section is the single source of truth for open problems

## Hard constraints
- Run `pytest tests/ -v` before every commit. Do not commit with failing tests.
- When adding functionality, add test cases to `tests/golden/test_cases.py` first.
- Verify spaCy dep trees empirically with `show_parse()` before assuming structure.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/james102387) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
