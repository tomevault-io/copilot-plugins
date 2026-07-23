## common-media-library

> Common Media Library is optimized for adoption by web video players — performance, tree-shakeability, developer experience, and documentation quality are the top priorities. General project instructions are in [AGENTS.md](AGENTS.md).

@AGENTS.md

# Claude Code Configuration

Common Media Library is optimized for adoption by web video players — performance, tree-shakeability, developer experience, and documentation quality are the top priorities. General project instructions are in [AGENTS.md](AGENTS.md).

## Skills

The following skills are available as slash commands:

- `/create-pr [base branch]` - Validate tests and docs, then create a GitHub PR with conventional commit formatting
- `/code-review [files or branch]` - Review code changes for performance, tree-shaking, bundle size, API ergonomics, and TypeScript best practices
- `/pr-feedback [PR number]` - Fetch unresolved PR review comments, validate them, plan fixes, implement, push, and resolve threads

## Agents

- **code-reviewer** - Specialized agent for code review, used by the `/code-review` skill

## Rules

- `.claude/rules/code-quality.md` - Automatically applied when editing files in `libs/**/*.ts`

---
> Source: [streaming-video-technology-alliance/common-media-library](https://github.com/streaming-video-technology-alliance/common-media-library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
