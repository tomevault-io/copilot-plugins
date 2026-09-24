## jira-cost-branches-prs

> Jira COST-#### branch and PR title conventions for this repo


# Jira branch and PR conventions (COST-####)

When work is tied to a Jira ticket `COST-####` (or the user references one):

| Item | Format |
|------|--------|
| Branch | `cost-####-short-kebab-slug` (lowercase, hyphens) |
| PR title | `[COST-####] Short English description` |
| Commit (recommended) | `[COST-####] Short English description` |

- Put `[COST-####]` at the **start** of the PR title, not `(COST-####)` at the end.
- PR body should link: `https://redhat.atlassian.net/browse/COST-####`
- Creating branches/PRs: `git checkout -b cost-####-slug` and `gh pr create --title "[COST-####] ..."`

Full table: [CONTRIBUTING.md](../../CONTRIBUTING.md).

Chore/docs without a ticket may use `chore/` or `docs/` branch prefixes instead.

---
> Source: [project-koku/koku-service-operator](https://github.com/project-koku/koku-service-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
