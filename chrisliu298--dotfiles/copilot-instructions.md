## dotfiles

> Personal dotfiles and AI agent configurations for macOS with zsh. Managed by `dotfiles.sh` (symlinks to `~`).

# CLAUDE.md

Personal dotfiles and AI agent configurations for macOS with zsh. Managed by `dotfiles.sh` (symlinks to `~`).

> This is the **project-level** CLAUDE.md. Global user instructions are in `agents/claude/CLAUDE.md`.

## Structure

```
dotfiles/
├── dotfiles.sh              # Single entrypoint: setup
├── shell/                   # Zsh config (.zshrc, .zshenv, .aliases, .functions, .p10k.zsh)
├── .config/                 # App configs (Neovim, tmux, btop)
├── .claude/skills/          # Project-local skills (dotfiles-only)
└── agents/
    ├── claude/              # Claude Code (~/.claude/) — CLAUDE.md, settings (copied), keybindings, hooks, statusline
    ├── codex/               # Codex (~/.codex/) — AGENTS.md
    └── extensions/skills/   # Global skills (SKILL.md per skill)
```

## Setup

```bash
./dotfiles.sh  # Initialize submodules, symlink files, verify links
```

## Conventions

- **Shell load order**: `shell/.zshenv` (platform detection, env, PATH) → `shell/.zshrc` (plugins, sources `.aliases` + `.functions`)
- **Modern tools**: `fd` → find, `rg` → grep, `delta` → diff, `zoxide` → z (installed via Zinit, macOS ARM)
- **Platform detection**: `IS_MACOS` set in `shell/.zshenv`, used for macOS-only code
- **Python**: Use `uv` for virtual environments (`sv`, `us`, `ua`, `upi` aliases)
- **Themes**: Ghostty (GitHub Dark), btop (GitHub Dark)

<important if="you are editing or creating skills">

## Editing Skills

Never edit in `~/.claude/skills/` or `~/.codex/skills/` — those are symlinks. Check `agents/extensions/README.md` for source.

- **Own skills** (`agents/extensions/skills/`): Edit in this repo — this is the single source of truth. Published GitHub repos are synced copies via `./dotfiles.sh publish`.
- **Third-party skills** (cloned from GitHub): Edit in the source repo or fork
- **Best practices**: Read `agents/extensions/references/skill-best-practices.md` before creating or improving skills

Run `./dotfiles.sh` after changes to re-sync symlinks.

</important>

<important if="you are adding a new extension or skill to dotfiles">

## Adding Extensions

Managed by `SKILLS` table in `dotfiles.sh` (git-clone + symlink, no `npx skills`):

- **Local**: Add `<name>/SKILL.md` under `agents/extensions/skills/`, run `./dotfiles.sh`
- **Upstream**: Add `name|owner/repo/subpath|agents` entry to `SKILLS` table
- **Agent-specific**: Separate table entries per agent (e.g., `pdf` has different sources for claude vs codex)
- **Manual skills**: Add name to `MANUAL_SKILLS` array — skipped during auto-install, toggled with `./dotfiles.sh enable/disable <name>`
- **Project-local skills**: Place in `.claude/skills/<name>/` — available only in this repo, not globally
- **Install/update all**: `./dotfiles.sh`

Single SKILL.md per skill works in both agents. Include Claude-specific frontmatter (`allowed-tools`, `user-invocable`) — Codex ignores unknown keys.

</important>

<important if="you are modifying CLAUDE.md, AGENTS.md, README.md, or project structure">

## Maintenance

- **Sync instructions**: `agents/claude/CLAUDE.md` and `agents/codex/AGENTS.md` share working principles — update both when changing either
- **Update docs**: After structural changes, check if `README.md`, `CLAUDE.md`, or `AGENTS.md` reference affected paths

</important>

## Not Backed Up

OAuth tokens, command history, local settings, per-project data, and cache files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chrisliu298) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
