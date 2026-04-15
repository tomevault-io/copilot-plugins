## claude-session-summary

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Claude Code plugin that automatically generates session summaries when Claude Code stops. Summaries are stored per-project under `~/.local/share/claude-sessions/{PROJECT_NAME}/` and viewable via tmux popup (`prefix + S`) using glow for markdown rendering.

## Commands

```bash
make install      # Add marketplace, install plugin via claude CLI
make uninstall    # Uninstall plugin and remove marketplace
make clean        # Same as uninstall but suppresses errors
make reinstall    # clean + install
```

## Architecture

The plugin is a single zsh script (`scripts/session-summary.zsh`) triggered by a Claude Code Stop hook defined in `.claude-plugin/plugin.json`.

**Execution flow:**
1. Stop hook fires with 5-second timeout
2. Script reads hook JSON input (session_id, cwd) from stdin with 0.5s timeout
3. Immediately forks to background (`&!`) so it survives the hook timeout
4. Calls `claude --continue --model haiku --output-format=json --disableAllHooks` to generate a summary
5. Parses JSON response with Python (`strict=False` handles control chars in output)
6. Writes `{SESSION_ID}.md` and updates `latest.md` symlink in the project directory
7. Logs cost/token data to `summary.log`

**Key design decisions:**
- The background fork is critical -- without it, the 5-second hook timeout kills the API call
- `--disableAllHooks` prevents recursive hook invocation
- Python is used over jq for JSON text extraction because Claude's output can contain unescaped control characters
- The hook returns 0 immediately regardless of background work outcome

## Runtime Dependencies

- zsh, jq, python3, Claude Code CLI
- Optional: glow (markdown rendering in tmux popup, falls back to cat)

## File Layout

- `.claude-plugin/plugin.json` -- Hook registration (Stop hook, 5s timeout)
- `.claude-plugin/marketplace.json` -- Plugin marketplace metadata
- `scripts/session-summary.zsh` -- All plugin logic (96 lines)
- `Makefile` -- Install/uninstall/clean/reinstall targets
- `.github/workflows/validate.yml` -- CI plugin validation on push/PR

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cosgroveb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cosgroveb)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
