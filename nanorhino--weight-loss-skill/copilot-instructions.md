## weight-loss-skill

> A collection of AI nutritionist skills — diet tracking, exercise planning,

# CLAUDE.md

## Project: weight-loss-skill

A collection of AI nutritionist skills — diet tracking, exercise planning,
meal planning, weight tracking, emotional support, and more. Each skill
is a self-contained module with its own SKILL.md.

## Required Reading

Before modifying any skill or data structure, read `docs/CONVENTIONS.md`.

## Key Rules

- **No language selection logic in skills** — `USER.md > Language` is the sole authority for reply language. Never add "reply in the user's language" or similar directives. See `docs/CONVENTIONS.md` §10.
- **No self-delivery wrappers in cron job payloads** — The "don't use exec/message to send" rule lives in `notification-composer` SKILL.md, not in each cron job's `payload.message`. Keep payload text to just the task instruction (e.g., `"Run notification-composer pre-send checks for breakfast."`). See `docs/CONVENTIONS.md` §11.
- Transactional/structured data → JSON under `data/`
- Profile-like content that changes slowly → Markdown at workspace root
- Group related data into one file per domain
- Use existing scripts for data access when available
- File naming: lowercase, hyphen-separated
- No dots in filenames as namespace separators
- Each data file has an owning skill — check before modifying its schema
- New fields in existing JSON files must be backward-compatible

## Project Structure

- Each skill lives in its own directory with a `SKILL.md`
- Shared scripts live in each skill's `scripts/` directory
- `docs/` contains project-level documentation
- Agent workspaces follow the layout defined in `docs/CONVENTIONS.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NanoRhino) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
