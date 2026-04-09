## cca

> This is a learning toolkit for terminal beginners. It lives in a directory the student chose during setup (default `~/cca`).

# CCA — Claude Code Architects

This is a learning toolkit for terminal beginners. It lives in a directory the student chose during setup (default `~/cca`).

## How this works

- `prompts/todo/` contains numbered prompt files pushed by the instructor.
- `.completed` (local, gitignored) lists prompt filenames that have already been run.
- On every session, check `prompts/todo/` for any prompts NOT listed in `.completed`.
- Run pending prompts in numeric order (001 before 002, etc).
- After successfully completing a prompt, append its filename to `.completed`.
- Never re-run a completed prompt unless the student explicitly asks.

## Prompt file format

Each prompt file in `prompts/todo/` is a self-contained instruction set. Read the entire file, then follow its instructions. Prompts may ask you to:
- Generate or update files in this directory
- Modify the student's shell config (rc file)
- Detect their environment and adapt accordingly

## Environment detection

When a prompt requires system-specific behavior, detect the environment:
```
uname -s          → Darwin (mac), MINGW/MSYS (windows/git-bash), Linux
echo $SHELL       → zsh or bash (determines rc file)
echo $TERM        → color support (if "dumb", no ANSI colors)
echo $LANG        → UTF-8 support
tput lines/cols   → terminal size
```

## Skills

Skills live in `.claude/skills/<name>/SKILL.md` with YAML frontmatter (`name`, `description`). Claude auto-loads them when the description matches the user's request, or they can be invoked directly with `/<name>`.

Available skills in this project:
- `/update` — check for new prompts from instructor, run them, then offer customization

## Key rules

- This is for beginners. Explain things simply. Don't overwhelm.
- Never delete or overwrite the student's local changes without asking.
- The `help.sh` file is local and gitignored — it's generated per-system, not tracked.
- If something fails, explain what went wrong in plain English and offer to fix it.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zenaitutoring)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zenaitutoring)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
