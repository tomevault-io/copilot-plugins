## trading-bot

> This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

# Agent Instructions

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

## IMPORTANT: Beads Configuration

The beads database for this rig is stored externally due to a Dolt embedded mode bug.
**You MUST set BEADS_DIR before running any bd command:**

```bash
export BEADS_DIR=/Users/hal/gt/.beads-data/tradingbot/.beads
```

Set this at the start of every session. All bd commands below require this env var.

## Quick Reference

```bash
export BEADS_DIR=/Users/hal/gt/.beads-data/tradingbot/.beads
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --status in_progress  # Claim work
bd close <id>         # Complete work
bd sync               # Sync with git
```

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd sync
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DreadPirateRobertz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DreadPirateRobertz)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
