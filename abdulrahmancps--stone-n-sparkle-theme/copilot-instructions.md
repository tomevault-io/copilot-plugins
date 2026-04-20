## stone-n-sparkle-theme

> Source of truth for how Cursor should work in the Stone-N-Sparkle theme repo — branch workflow, PR readiness, production safety


# Cursor Rules (Stone-N-Sparkle Theme)

Use this file as the **source of truth** for how Cursor should work in this repository.

---

## 1) Mission
Cursor must help developers ship safe, reviewable theme changes without breaking production.

---

## 2) Non-Negotiable Rules
1. Never develop directly on `main`.
2. Always use a branch per task:
   - `feature/<short-name>` for new work
   - `fix/<short-name>` for bug fixes
3. Keep changes minimal and scoped to the assigned task.
4. Merge to `main` only through Pull Requests (PRs).
5. Never commit secrets, keys, DB exports, or backups.
6. Do not modify deployment workflows unless explicitly asked.

If a request conflicts with these rules, Cursor must warn and propose a safe alternative.

---

## 3) Required Branch Workflow
Before coding:
```bash
git checkout main
git pull origin main
git checkout -b feature/<short-name>
```

During coding:
- Commit in small logical chunks.
- Use clear commit messages.

Before PR:
```bash
git fetch origin
git merge origin/main
```
Resolve conflicts in the feature branch, then test again.

---

## 4) PR Readiness Checklist
Cursor should confirm all relevant checks before suggesting merge:
- [ ] Homepage loads locally
- [ ] Shop page loads locally (if relevant)
- [ ] Product page loads locally (if relevant)
- [ ] No PHP fatal errors / blank pages
- [ ] ACF changes include updated `acf-json/` files (if relevant)

---

## 5) Production Safety Rules
- `main` is production-bound.
- If production breaks after merge:
  1. Stop further merges.
  2. Run rollback workflow.
  3. Verify production recovery.
  4. Fix forward on `fix/...` branch and open PR.

---

## 6) Collaboration Rules (Parallel Team Work)
- Announce when editing shared core files (`functions.php`, `header.php`, `footer.php`).
- Prefer small PRs over large refactors.
- If conflicts appear, re-sync from `origin/main` and resolve in branch.

---

## 7) File Hygiene (Never Commit)
- `wp-config.php`
- `.env` and `.env.*`
- `*.wpress`, `*.sql`, `*.zip`
- `wp-content/uploads/`
- private keys, tokens, credentials

---

## 8) Plain-English GitHub Terms (for non-experts)
- **Branch**: your private work lane.
- **feature branch**: lane for a new change.
- **fix branch**: lane for a bug fix.
- **PR (Pull Request)**: request to merge your lane into `main`.
- **merge conflict**: Git cannot auto-combine changes; you must choose final code.
- **rollback**: restore last known good production backup.

---

## 9) Recommended Prompt Template for Cursor
Use this when starting a task:

> Work only on branch `<branch-name>`.  
> Follow `CURSOR_RULES.md`, `README.md`, and `CONTRIBUTING.md`.  
> Keep diff minimal and limited to `<target files/feature>`.  
> Do not edit deployment workflows.  
> Never add secrets.  
> Before finishing: run checks, summarize changed files, and propose commit message + PR summary.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AbdulrahmanCPS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
