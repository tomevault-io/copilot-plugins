## git-pr-workflow

> Rules for creating Git branches and pull requests


# Git PR Workflow

## Always create a new PR from a fresh branch off main

When the user asks you to push a fix or create a PR:

1. **Always check if the previous PR has already been merged** before pushing to an existing branch.
2. **If the previous PR was already merged**, do NOT push additional commits to the old branch. Instead:
   - `git checkout main && git pull origin main`
   - Create a **new branch** from the updated `main`
   - Apply the changes on the new branch
   - Push and create a **new PR**
3. **If the previous PR is still open**, you may push additional commits to the existing branch.

This avoids pushing commits to stale/merged branches where they will have no effect on production.

## PR creation checklist

- Always target `main` as the base branch
- Always add reviewers when specified by the user
- Write a clear summary explaining what changed and why
- Include a test plan

## No fake data

Never create fake, mock, or placeholder data in any branch, PR, or pipeline run. All data must come from real sources (data.gouv.fr, production database, etc.).

---
> Source: [dataforgoodfr/13_eclaireur_public](https://github.com/dataforgoodfr/13_eclaireur_public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
