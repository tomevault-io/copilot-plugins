## auto-review-claudemcp

> - NEVER run `git push`, `git commit`, or `git add` unless the user explicitly asks to commit or push.

# Project Rules

## Restrictions

- NEVER run `git push`, `git commit`, or `git add` unless the user explicitly asks to commit or push.
- NEVER modify, fix, or patch code in the target repository being reviewed. This tool is read-only — it only reviews PRs, it does not fix them.
- When reviewing a PR, only report findings. Do not suggest auto-applying fixes to the target repo.

## Purpose

This is a PR review MCP server. Claude's role is to:
1. Fetch and analyze PR diffs via the MCP tools
2. Report issues, suggestions, and observations
3. Generate review reports (HTML or text)

Claude does NOT have access to the target repository's codebase — only to the diffs returned by the GitHub API.

---
> Source: [IskanderAl/Auto-Review-ClaudeMCP](https://github.com/IskanderAl/Auto-Review-ClaudeMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
