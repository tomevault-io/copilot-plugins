## nitpicker

> A Claude Code skill for AI-powered pull request review.

# nitpicker

A Claude Code skill for AI-powered pull request review.

## Usage

```
/review-pr <diff-url>
```

Example:
```
/review-pr https://patch-diff.githubusercontent.com/raw/owner/repo/pull/4.diff
```

The diff URL can be any publicly accessible unified diff. On GitHub, append `.diff` to a PR URL.

## How it works

1. Fetches the diff via `curl`
2. Parses changed files and pairs each source file with its test file (if both are in the diff)
3. **Phase 1** — spawns one reviewer agent per file in parallel; each agent reviews its file independently
4. **Phase 2** — agents run again with full visibility into each other's findings; cross-file concerns are surfaced
5. Final report aggregates all reviews with blocker/suggestion/nit counts and an overall verdict

## Skill location

`.claude/skills/review-pr/SKILL.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nizarmah)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nizarmah)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
