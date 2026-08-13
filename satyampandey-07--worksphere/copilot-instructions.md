## worksphere

> Before creating any PR, verify the 24h window has passed since the last PR:

# WorkSphere Agent Workflow

## PR 24-Hour Window Check

Before creating any PR, verify the 24h window has passed since the last PR:

```powershell
# Check current UTC time
Get-Date -Format "yyyy-MM-dd HH:mm:ss UTC"

# Get last PR creation time
$lastPr = gh pr list --author Prathvikmehra --state merged --limit 1 --json createdAt --jq '.[0].createdAt'
Write-Output "Last PR: $lastPr"
```

If `current time < last PR time + 24h`, DO NOT create PRs. Wait until the window opens.

## Issue Creation Flow

1. Create issue with `gh issue create`
2. Immediately comment `/claim` to auto-assign
3. Verify assignment: `gh issue view <number> --json assignees`

---
> Source: [SatyamPandey-07/WorkSphere](https://github.com/SatyamPandey-07/WorkSphere) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
