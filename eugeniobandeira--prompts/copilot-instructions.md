## prompts

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository

This repository (`prompts`) is a centralized collection of Claude Code custom slash commands for software development workflows. Clone it and run the install script to make all commands available globally in your terminal.

## Installation

**Windows (PowerShell):**
```powershell
git clone <repo-url>
cd prompts
.\install.ps1
```

**Mac / Linux:**
```bash
git clone <repo-url>
cd prompts
bash install.sh
```

The scripts copy all files from `.claude/commands/` to `~/.claude/commands/`, making the commands available as slash commands in any project opened with Claude Code.

## Available Commands

| Command | Purpose |
|---|---|
| `/write-spec` | Write a technical spec before coding a new feature |
| `/write-plan` | Translate a spec into a step-by-step implementation plan |
| `/write-code` | Execute an implementation plan against the codebase |
| `/review-code` | Audit an implementation against spec, plan, and architecture rules |
| `/verify` | Run the full verification pipeline before any commit or PR |
| `/commit` | Stage and commit using Conventional Commits standard |
| `/create-pr` | Generate a structured PR and open it via GitHub CLI |
| `/map-codebase` | Analyze an existing codebase and produce architecture documentation |

## Recommended Workflow

```
/write-spec → /write-plan → /write-code → /verify → /review-code → /commit → /create-pr
```

## Adding or Updating Commands

1. Add or edit `.md` files in `.claude/commands/`
2. Re-run the install script to update the global installation
3. Commit and push so the team gets the update on next `git pull` + install

---
> Source: [eugeniobandeira/prompts](https://github.com/eugeniobandeira/prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
