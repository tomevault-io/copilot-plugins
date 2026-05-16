## acm-infra

> All git commits in this repository MUST include a Signed-off-by line using the git-configured user details.

# Project Instructions for Claude

## Git Commits

All git commits in this repository MUST include a Signed-off-by line using the git-configured user details.

The format should be:
```
Signed-off-by: $(git config user.name) <$(git config user.email)>
```

This should appear in the commit message footer, before any Co-Authored-By lines.

**Note:** You do not need permission to run `git config user.name` or `git config user.email` - these are read-only commands that retrieve necessary information for commit messages.

## Example Commit Message

```
Add new feature

This commit adds a new feature to improve the workflow.

Signed-off-by: Nathaniel Graham <ngraham@redhat.com>
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

---
> Source: [stolostron/acm-infra](https://github.com/stolostron/acm-infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
