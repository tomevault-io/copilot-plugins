## atom2api

> - For every code change made by Codex in this repository, Codex must use and follow `$git-commit-quality` before considering the work complete.

# Codex Project Rules

## Commit And Push Requirement

- For every code change made by Codex in this repository, Codex must use and follow `$git-commit-quality` before considering the work complete.
- Codex must inspect the actual diff, keep the commit focused, stage only the files or hunks that belong to the current task, and preserve unrelated user changes.
- Codex must run the smallest relevant verification command it can infer from the repository before committing. If no reasonable verification exists, Codex must report that explicitly.
- Codex must commit the completed change with the repository's existing commit style, or with scoped Chinese Conventional Commit format when no stronger local style exists.
- After a successful commit, Codex must push the commit to the current branch's configured upstream remote.
- If the current branch has no upstream, push authentication fails, verification fails, or the working tree contains ambiguous unrelated changes, Codex must stop and report the blocker instead of bypassing this rule.
- Codex must not commit or push unrelated files, generated noise, secrets, local environment files, or user changes that are outside the current task.

---
> Source: [cnluminous/Atom2Api](https://github.com/cnluminous/Atom2Api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
