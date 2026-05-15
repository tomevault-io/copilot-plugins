## kalypso-claude-workflow

> These are my personal Claude Code defaults. They live in `~/.claude/CLAUDE.md` and apply to every session. Per-project `CLAUDE.md` files override these where needed.

# Global Workflow Rules

These are my personal Claude Code defaults. They live in `~/.claude/CLAUDE.md` and apply to every session. Per-project `CLAUDE.md` files override these where needed.

## Plan mode first

For any task touching more than two files, start in plan mode (`shift+tab` twice) and get sign-off before editing. If the approach isn't obvious from a one-line prompt, plan it.

## Prefer focused subagents over main context

- Use the `pr-reviewer` subagent for any review task — keeps main context clean and gives an independent read.
- Subagents run in their own context, so the review doesn't eat my main session's tokens.

## Commit hygiene

- One commit per file for multi-file changes. Use `/commit-split`.
- Never use `--no-verify`. Fix the hook failure instead.
- Never `git push --force` to shared branches — warn and stop if asked.

## Non-destructive by default

- Risky operations (`rm`, `chmod`, `docker`, `npm install`, `git push`, `git commit`) require confirmation via the `ask` permission tier.
- Secret files (`.env*`, `~/.ssh/**`, `~/.aws/**`) are read-denied at the global level. If a specific value is needed, paste it — don't let the model read the whole file.

## Verification before "done"

Don't claim a task is complete until:
1. Typecheck passes (if the project has one)
2. Relevant tests pass
3. Lint passes

Use `/pr-ready` to batch-run these.

## Useful flags and commands

- `claude -w` — start in a new git worktree for parallel work without branch-switching cost.
- `claude --bare` — ~10× faster SDK startup; skips CLAUDE.md + settings. Use for one-shot scripted calls.
- `claude --add-dir /path/to/second/repo` — access two repos in one session instead of context-switching.
- `/powerup` — sweep the 10 built-in lessons once; surfaces features that are easy to miss.
- `/branch` to fork a session; `/btw` for side queries without breaking the main flow.
- `/rewind` / `Esc-Esc` to undo Claude's last change.

---
> Source: [Kalypsokichu-code/kalypso-claude-workflow](https://github.com/Kalypsokichu-code/kalypso-claude-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
