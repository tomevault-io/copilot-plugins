## kudu

> Instructions for all AI sub-agents (Claude Code agents, worktree agents, etc.) working on this codebase.

# Agents

Instructions for all AI sub-agents (Claude Code agents, worktree agents, etc.) working on this codebase.

## Commit Conventions

Always use [Conventional Commits](https://www.conventionalcommits.org/). Format:

```
<type>(<scope>): <short summary>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`.

Breaking changes must include `!` after the type/scope.

## Before Committing

- Run `npm test` to ensure all tests pass.
- Run `npm run validate:rules` if rule JSON files were changed.

## Code Style

- Follow existing patterns in the codebase.
- Keep PRs focused — one logical change per branch.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AdventDevInc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AdventDevInc)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
