## anti-sycophant-ai-agent-skills

> This file is for AI coding agents (Claude Code, Cursor, Copilot, Gemini, OpenCode, Codex, Aider, Cline, anything else) working *inside this repository*. For end-user installation of these skills in your own projects, see [docs/getting-started.md](docs/getting-started.md).

# AGENTS.md

This file is for AI coding agents (Claude Code, Cursor, Copilot, Gemini, OpenCode, Codex, Aider, Cline, anything else) working *inside this repository*. For end-user installation of these skills in your own projects, see [docs/getting-started.md](docs/getting-started.md).

## What this repo is

A pack of three skills that get AI assistants to push back on unvalidated product ideas instead of agreeing by default. The skills are plain Markdown with YAML frontmatter — portable across any LLM that accepts a system prompt.

## Repo layout

```
skills/                     One directory per skill, each containing SKILL.md.
  prove-the-premise/        Pressure-test idea premise before building.
  hobby-or-business/        Separate project enthusiasm from business reality.
  one-real-conversation/    Reject validation theater; demand one honest human conversation.

.claude/commands/           Claude Code slash commands.
.gemini/commands/           Gemini CLI slash commands.
.claude-plugin/             Claude Code plugin manifests.
docs/                       Per-tool setup guides.
```

## Rules for agents editing this repo

1. **Skills live at `skills/<name>/SKILL.md`.** Kebab-case directory name. The directory name must match the `name:` field in the frontmatter exactly.
2. **Frontmatter is non-negotiable.** Every skill needs `name` and `description`. The description must include both `Use when` triggers and a `Do NOT use when` off-switch. Skills without an off-switch are bugs.
3. **One file per skill: `SKILL.md`, and nothing else inside the directory.** No `scripts/`, no supporting `.md` files, no references. If a skill outgrows ~500 lines, that's a sign the skill is trying to do too much — split it into two skills with distinct triggers, don't add scratch files. Long shared reference material across multiple skills would go in a top-level `references/` directory; so far nothing has needed one.
5. **Don't add tooling without being asked** — no test runners, validation scripts, CI configs, or hooks. The repo is intentionally minimal.
6. **Preserve tone.** The skills are blunt on purpose. Don't soften them. But also don't sharpen them into cruelty — that's a tone regression.
7. **Cross-reference rather than duplicate.** If one skill needs guidance from another, link to it by name.

## How the skills are meant to fire

Skills auto-trigger off the `description` field's `Use when` clauses. The model loads a skill when a conversation matches its triggers, not because of any orchestration layer.

There are also two slash commands (`/validate-idea`, `/reality-check`) as explicit entry points for Claude Code and Gemini CLI. They invoke one or more skills directly when the user wants to apply the framework on demand instead of waiting for trigger phrases.

## Format reference

See [docs/skill-anatomy.md](docs/skill-anatomy.md) for the full format spec and [CONTRIBUTING.md](CONTRIBUTING.md) for the quality bar.

---
> Source: [machinesoul11/anti-sycophant-ai-agent-skills](https://github.com/machinesoul11/anti-sycophant-ai-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
