## gmail2trello

> Read all of AGENTS.md and follow and read all its links and do everything it says.

Read all of AGENTS.md and follow and read all its links and do everything it says.

# Editor Settings for Consistent Indentation
editor.tabSize: 2
editor.insertSpaces: true
editor.detectIndentation: false

# Background Agent Branch Cleanup
When a background agent task completes:
- If the branch has no commits beyond the base branch: delete both local and remote branches
- Check with: git rev-list --count HEAD ^main (if 0, delete the branch)
- Delete with: git checkout main && git branch -D <branch> && git push origin --delete <branch> 2>/dev/null || true

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/appliedmedia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
