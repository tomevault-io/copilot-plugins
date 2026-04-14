## claude-code-tool-kit

> > Universal Claude Code configuration — stack-agnostic, upgradable, shared across all projects.

# Project Instructions

> Universal Claude Code configuration — stack-agnostic, upgradable, shared across all projects.
> Detailed rules are in `.claude/rules/` and auto-load into context.

## Environment Detection

Current directory: !`pwd`
Project files: !`ls -1 2>/dev/null | head -20`
Git branch: !`git branch --show-current 2>/dev/null || echo "not a git repo"`
Language hints: !`ls package.json requirements.txt Pipfile pyproject.toml go.mod Cargo.toml *.csproj *.sln Gemfile build.gradle pom.xml composer.json mix.exs deno.json 2>/dev/null`

---

## Core Principles

- Follow the existing patterns and conventions already present in THIS project
- Detailed standards are in `.claude/rules/` — they auto-load per session
- When rules conflict with project conventions, project conventions win

## File Operations
- Verify directories exist before creating files
- Prefer editing specific sections over rewriting entire files
- Respect .gitignore — never modify ignored file categories
- Follow existing naming conventions in the project

## Communication
- Be concise but thorough — explain "why" not just "what"
- Flag potential issues and trade-offs proactively
- Ask clarifying questions rather than assuming

## Boundaries
- Do not refactor files outside the current task scope
- Do not install new packages without confirming with the user
- Do not modify lock files manually
- Do not overwrite .env files or secrets
- Do not make architectural changes without discussion
- Do not delete tests, even if currently failing

---

## Project-Specific Configuration

> **Customize this section per project**, or create `CLAUDE.local.md` for personal notes.

### Stack
<!-- Language, framework, database, test runner, package manager -->

### Build & Dev Commands
<!-- dev server, build, test, lint, type check -->

### Architecture Notes
<!-- Key directories, patterns, data flow -->

### Key Conventions
<!-- Project-specific conventions beyond the universal rules -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/viknesh20-20)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/viknesh20-20)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
