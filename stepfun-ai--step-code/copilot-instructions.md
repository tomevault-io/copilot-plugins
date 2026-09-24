## step-code

> - Keep the Git author and committer as the human contributor's verified GitHub identity.

# Agent instructions

## Commit attribution

- Keep the Git author and committer as the human contributor's verified GitHub identity.
- AI tools may assist with implementation, but they must not be added as commit co-authors.
- Do not add `Co-authored-by` or `Co-Authored-By` trailers for Claude, Claude Code, Codex, ChatGPT, OpenAI, Anthropic, or another AI client.
- Do not mention the client used to create a change in the commit message solely for attribution.
- Before creating or amending a commit, inspect its metadata and remove any AI-client attribution trailer.

This policy applies to commit metadata. Documentation may mention AI products when that is relevant to the product or feature being documented.

---
> Source: [stepfun-ai/Step-Code](https://github.com/stepfun-ai/Step-Code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
