## hoangduy0308

> - Keep this repository simple. It is a GitHub profile README repository, not an application codebase.

# Agent Instructions

## General

- Keep this repository simple. It is a GitHub profile README repository, not an application codebase.
- Do not use repo-local issue tracker tooling in this repo.
- When spawning a new agent, always set `fork_context = false`.
- If the same error happens twice, research 3-5 possible fixes, choose the most efficient one, and implement it.

## Workflow

- Make small, focused changes.
- For README/profile edits, work pragmatically; do not force TDD.
- Use `srcwalk` before broad manual reading when doing code-intelligence work and the CLI is available.
- Use `rg` for exact text search and small known-file checks.

## Commit Convention

Use Conventional Commits:

```text
<type>(<scope>): <subject>
```

Common types: `docs`, `fix`, `feat`, `chore`, `refactor`, `test`, `style`, `ci`.

Examples:

```bash
git commit -m "docs(readme): simplify profile"
git commit -m "chore(repo): remove unused workflow"
```

## Completion

Before finishing a change:

```bash
git status
git diff --check
git push
```

Work intended for review is not finished until it has been pushed successfully.

---
> Source: [hoangduy0308/hoangduy0308](https://github.com/hoangduy0308/hoangduy0308) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
