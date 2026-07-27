## docs-book-cloudfoundry

> Ensure all doc-set repos are on master before making any edits


# Switch all repos to master before editing

**This is the first action for every task in this doc set, before any file read, plan, or edit.**

If you have not yet confirmed that every affected repo is on the `master` branch, stop and do it now:

- PowerShell (from the book repo): `scripts/ensure-master.ps1`
- Bash (from the book repo): `scripts/ensure-master.sh`

Or check and switch each repo manually:

```powershell
git -C <repo-path> rev-parse --abbrev-ref HEAD   # must return "master"
git -C <repo-path> checkout master                # switch if not already on master
```

Do not write or edit any file until every affected repo is confirmed on `master`.

---
> Source: [cloudfoundry/docs-book-cloudfoundry](https://github.com/cloudfoundry/docs-book-cloudfoundry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
