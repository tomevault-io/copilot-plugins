## focus-group

> Agent Skills package that clones a user's customers from real sales-call transcripts and runs them as a standing focus group. Installable across Claude Code (most common host), Codex, Cursor, and other [Agent Skills](https://agentskills.io) hosts. Pure-stdlib Python for preflight/scaffolding; the clustering, dossiers, and panel runs are agent work driven by SKILL.md.

# focus-group skill

Agent Skills package that clones a user's customers from real sales-call transcripts and runs them as a standing focus group. Installable across Claude Code (most common host), Codex, Cursor, and other [Agent Skills](https://agentskills.io) hosts. Pure-stdlib Python for preflight/scaffolding; the clustering, dossiers, and panel runs are agent work driven by SKILL.md.

## Structure

- `skills/focus-group/SKILL.md` — canonical skill contract the model reads when `/focus-group` fires. Source of truth for behavior across every host: setup wizard, panel protocol, persistence rules.
- `skills/focus-group/scripts/setup.py` — preflight (`--check` / `--json`), builder (bare), `--mark-complete`.
- `skills/focus-group/scripts/build-skill.sh` — builds `dist/focus-group.skill` for claude.ai upload (dev-only).
- `hooks/` — Claude Code SessionStart setup-status hook (Claude Code only).
- `.claude-plugin/` — `plugin.json` + `marketplace.json` (Claude Code plugin + marketplace).
- `CLAUDE.md` → `@AGENTS.md` — generic-agent entry point.
- `tests/` — pytest suite (temp-HOME fixtures; no network).

## Orientation

- The product is the slash-command-invoked skill (`/focus-group <stimulus>`), not a CLI. `scripts/setup.py` only does preflight and scaffolding — all persona/panel intelligence lives in SKILL.md and must work across every harness, not just Claude Code.
- **The skill is one self-contained folder: `skills/focus-group/`.** SKILL.md and `scripts/` are siblings inside it — do NOT move either to the repo root, or non-Claude installers will copy SKILL.md without the scripts.
- **Path resolution is harness-agnostic.** SKILL.md resolves `SKILL_DIR` as the directory of the SKILL.md the model just Read. Do NOT reintroduce `${CLAUDE_SKILL_DIR}` — it is unset on other hosts.
- **Runtime state lives outside the skill:** the group (panel dossiers, sessions) in `~/Documents/FocusGroup/` (or `SAVE_DIR`), config in `~/.config/focus-group/.env`. Never clobber either — append-only.
- **Disagreement is the product.** Nothing in the panel protocol may rank responses, pick a winner, or synthesize consensus. Keep the moderator opinion-free.
- **No `commands/` wrapper.** `/focus-group` is derived from SKILL.md frontmatter (`name` + `user-invocable: true`).

## Install surfaces

| Surface | Install |
|---------|---------|
| Claude Code | `/plugin marketplace add bradautomates/focus-group` then `/plugin install focus-group@focus-group` |
| Codex / Cursor / other hosts | `npx skills add bradautomates/focus-group -g` |
| claude.ai (web) | upload `dist/focus-group.skill` (built by `skills/focus-group/scripts/build-skill.sh`) |

## Commands

```bash
# Tests (stdlib + pytest; no network)
.venv/bin/python -m pytest tests/ -q      # or: python3 -m pytest tests/ -q

# Build the claude.ai upload bundle (archives skills/focus-group/ as the bundle root)
bash skills/focus-group/scripts/build-skill.sh   # → dist/focus-group.skill

# Dev: mirror the working tree into the installed Claude Code plugin cache
./dev-sync.sh                              # --dry-run to preview
```

## Rules

- Keep the version in sync across `skills/focus-group/SKILL.md` (frontmatter) and `.claude-plugin/plugin.json` when cutting a release.
- Releasing: tag `vX.Y.Z` and push the tag; `.github/workflows/release.yml` runs tests, builds `dist/focus-group.skill`, and attaches it to the GitHub release. Update `CHANGELOG.md` first.
- Never commit real transcripts, personas, or `.env` contents; runtime state lives under `~/Documents/FocusGroup/` and `~/.config/focus-group/.env` (mode `0600`).
- Panel seats are composites of 3+ real people — never weaken that constraint or allow a real named person as a seat.

---
> Source: [bradautomates/focus-group](https://github.com/bradautomates/focus-group) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
