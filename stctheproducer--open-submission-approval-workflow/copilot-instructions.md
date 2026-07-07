## pull-request-template

> Require repo PR templates when creating or updating pull requests.


# Pull request templates

Before creating or materially updating a pull request, check the repository for a PR template and use it as the body structure.

## Find the template first

Search these paths in order until one exists:

1. `.github/PULL_REQUEST_TEMPLATE.md`
2. `.github/pull_request_template.md`
3. `.github/PULL_REQUEST_TEMPLATE/*.md` (use the default template if multiple)
4. `docs/pull_request_template.md`
5. `PULL_REQUEST_TEMPLATE.md` at the repository root

Read the template file before drafting the PR body. Do not invent a custom layout when the repo defines its own sections.

## Fill the template faithfully

- Keep every section and heading from the template.
- Replace placeholder comments with concrete content for the current change.
- Mark checklist items with `[x]` only when they are true for this PR.
- Leave optional sections in place when they do not apply; briefly note `N/A` instead of deleting template structure.
- Preserve repo-specific notes at the bottom of the template, including Conventional Commits title guidance.

## PR scope

- PR titles should follow Conventional Commits.
- A PR may contain multiple closely related vertical slices when they form one coherent review unit.
- Do not split work into extra PRs purely for process overhead when one branch and one PR make the review clearer.
- When a PR contains multiple slices, make the related issues, test coverage, and reviewer notes explicit in the PR body.

## Create the PR with `gh`

Use `gh pr create` or `gh pr edit` and pass the filled template through a heredoc body.

```bash
gh pr create --title "feat(workflow): add application review transitions" --body "$(cat <<'EOF'
<paste filled template here>
EOF
)"
```

Do not push a PR body that ignores an existing repository template.

---
> Source: [stctheproducer/open-submission-approval-workflow](https://github.com/stctheproducer/open-submission-approval-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
