## gh-merge

> When merging PRs with gh, use squash merge so main stays linear (one commit per PR).


# GitHub PR merge behavior

When merging pull requests using the `gh` CLI, **always use squash merge** so that `main` has a linear history (one commit per PR) and no merge commits.

Use:

```bash
gh pr merge <number> --squash
```

Do **not** use:

```bash
gh pr merge <number> --merge
```

This matches the repo preference for squash merging (configure in GitHub: Settings → General → Pull Requests → allow squash merging and default to squash merging).

---
> Source: [apt-bundle/apt-bundle](https://github.com/apt-bundle/apt-bundle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
