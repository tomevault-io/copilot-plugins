## github

> Use the connected GitHub MCP for issues, pull requests, reviews, and repository data


# GitHub goes through MCP

This is an **Apply Intelligently** rule: no `globs`, `alwaysApply` is false.

When the user asks about GitHub issues, PRs, reviews, branches, releases,
collaborators, or repo metadata:

1. Use the connected **GitHub** MCP. Do not guess issue numbers or PR state.
2. Call `get_me` first if owner or permission context is missing.
3. Use `list_*` for simple lists. Use `search_*` for filtered or natural-language queries.
4. Prefer `minimal_output: true` when the full payload is not needed.
5. If the GitHub MCP is disconnected, say so and stop.

Prefer the `github` skill for multi-step issue or PR workflows.

---
> Source: [damien-xai/cursor-rules-skills-mcp-hooks](https://github.com/damien-xai/cursor-rules-skills-mcp-hooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
