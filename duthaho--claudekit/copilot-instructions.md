## claudekit

> This is the claudekit plugin — a comprehensive toolkit for Claude Code with 44 skills, 20 agents, and an interactive setup wizard.

# Claudekit Plugin

This is the claudekit plugin — a comprehensive toolkit for Claude Code with 44 skills, 20 agents, and an interactive setup wizard.

## Plugin Structure

- `skills/` — 44 auto-triggered skills (invoked as `/claudekit:<name>`)
- `agents/` — 20 specialized agents (invoked as `claudekit:<name>`)
- `scripts/` — Hook scripts installed via `/claudekit:init`
- `skills/init/templates/` — Templates for rules, modes, hooks, and MCP configs

## Setup

After installing the plugin, run `/claudekit:init` to scaffold project-level configuration (rules, modes, hooks, MCP servers) into your project's `.claude/` directory.

## Skills

Skills auto-trigger based on context. Key categories:

- **Tech Stack**: languages, backend-frameworks, frontend, frontend-styling, databases, devops, testing
- **Domain**: openapi, owasp, playwright, error-handling, state-management, logging, caching, api-client
- **Patterns**: authentication, background-jobs, writing-concisely
- **Workflows**: feature-workflow, git-workflows, documentation, refactoring, performance-optimization, mode-switching, session-management
- **Methodology**: brainstorming, writing-plans, executing-plans, test-driven-development, systematic-debugging, verification-before-completion, and more

## Conventions

- Skills use YAML frontmatter with `name`, `description`, and optional `user-invocable`, `argument-hint`, `disable-model-invocation`
- Agents use markdown frontmatter with `name`, `description`, `model`, `tools`, `disallowedTools`
- Hook scripts follow "fail open" pattern — errors never block work
- Templates in `skills/init/templates/` are copied to the user's project, not loaded as plugin context

---
> Source: [duthaho/claudekit](https://github.com/duthaho/claudekit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
