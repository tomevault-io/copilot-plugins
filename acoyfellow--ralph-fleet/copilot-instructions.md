## ralph-fleet

> Ralph Loop that orchestrates Ralph Loops across a GitHub account/org.

# AGENTS.md

Ralph Loop that orchestrates Ralph Loops across a GitHub account/org.

---

## STOP CONDITION

All target repos have functioning Ralph infrastructure installed and verified.

---

## WHAT THIS REPO DOES

This is a **fleet orchestrator**. It:
1. Scans GitHub repos (personal or org)
2. Installs/updates Ralph infrastructure on each
3. Reports fleet status
4. Can trigger coordinated actions across repos

---

## ABSOLUTE RULES

1. **PRD only**: Work from `prd.json`
2. **One story per iteration**
3. **Never commit secrets**: GitHub tokens live in `.env` only
4. **Verify before commit**: `bun test && bun tsc`
5. **Dry-run first**: Destructive operations require `--dry-run` pass first

---

## LOCAL DEV

```bash
# Install
bun install

# Set GitHub token
cp .env.example .env
# Add: GITHUB_TOKEN=ghp_xxx (needs repo scope)

# Run commands
bun run scan          # List repos
bun run status        # Fleet status
bun run install <repo> # Install Ralph on one repo
bun run sync          # Sync config across fleet
```

---

## GITHUB API

- Auth: `GITHUB_TOKEN` env var
- Rate limit: 5000/hr authenticated
- Use octokit: `import { Octokit } from '@octokit/rest'`

---

## RALPH INFRASTRUCTURE (what gets installed)

Target repos get:
- `AGENTS.md` (from templates/)
- `scripts/ralph/prd.json` (initialized)
- `scripts/ralph/progress.txt` (empty)
- `.github/workflows/ralph.yml` (optional CI)

---

## KNOWN FOOTGUNS

- Don't overwrite existing AGENTS.md without `--force`
- Check for existing PRD stories before reinitializing
- Rate limiting: batch API calls, sleep between repos

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/acoyfellow)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/acoyfellow)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
