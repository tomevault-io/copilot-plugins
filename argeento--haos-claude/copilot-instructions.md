## haos-claude

> This is the **meta project** — the repository that builds and distributes haos-claude. Do NOT confuse it with `dist/.claude/CLAUDE.md` which is the end-user system prompt deployed to users' machines.

# CLAUDE.md — haos-claude development

This is the **meta project** — the repository that builds and distributes haos-claude. Do NOT confuse it with `dist/.claude/CLAUDE.md` which is the end-user system prompt deployed to users' machines.

## Project structure

```txt
├── dist/                    # Everything that gets deployed to users
│   ├── .claude/             # → copied to ~/.claude/
│   │   ├── CLAUDE.md        # End-user system prompt (Claude on HAOS)
│   │   ├── settings.json    # Claude Code permissions
│   │   └── skills/          # Skill files (6 skills)
│   ├── haos                 # CLI wrapper script (cmd/put/api/ws/start)
│   └── version.txt          # Current version (semver)
├── install.sh               # User runs this to install (downloads dist/ from GitHub)
└── README.md
```

## How it works

- Users run `install.sh` on their local PC
- It downloads `dist/` contents from GitHub raw to `~/.claude/`
- `dist/.claude/*` → `~/.claude/` (CLAUDE.md, settings.json, skills)
- `dist/haos` → `~/.claude/haos` (CLI wrapper)
- `dist/version.txt` → `~/.claude/version.txt`
- Creates `~/.claude/.env` with default config (if not exists)

The `haos` wrapper has 5 subcommands:

- `haos start` — launches Claude with session checks
- `haos cmd <command>` — SSH to HAOS
- `haos put <local> <remote>` — SCP to HAOS
- `haos api <METHOD> <ENDPOINT> [BODY]` — HA Core REST API
- `haos ws <TYPE> [JSON_DATA]` — HA WebSocket API (via ha_ws.py)

## Key conventions

- All HAOS commands in skills/CLAUDE.md use `haos cmd` prefix (not bare `ha` or `haos ha`)
- All REST API calls use `haos api` (not `ha-api` or raw `curl`)
- All WebSocket calls use `haos ws` (not direct `ha_ws.py`)
- `dist/.claude/settings.json` has only one permission: `Bash(haos *)`
- Language is stored in `.env` as `HAOS_LANGUAGE`, injected into CLAUDE.md by install script and auto-update
- `.env` is NEVER overwritten on reinstall (only created if missing)
- Version format: semver in `dist/version.txt`

## When editing skills or CLAUDE.md

- Every command that runs on HAOS must go through `haos cmd`
- Every REST API call must go through `haos api`
- Every WebSocket call must go through `haos ws`
- Never use bare `ha`, `ssh`, `scp`, or `curl` in examples
- Test that `install.sh` and `haos start` auto-update file lists match actual files in `dist/`

## Release workflow

1. Edit files in `dist/`
2. Bump `dist/version.txt`
3. Commit and push to `main`
4. Users get updates automatically via `haos start`

---
> Source: [Argeento/haos-claude](https://github.com/Argeento/haos-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
