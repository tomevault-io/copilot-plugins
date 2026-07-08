## toolset

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A Claude Code plugin monorepo. It publishes reusable extensions to the Claude Code plugin marketplace under the `mark-satterfield` namespace. Although it targets Claude Code, most — if not all — plugins work well with other agent harnesses, like Codex.

Everything ships as a **plugin**: `plugins/<name>/`. There is no longer a top-level `skills/` directory — skills now live inside the plugin that owns them. Loose, miscellaneous skills live in the catch-all `plugins/skills` plugin.

## Plugins

These 15 plugins are registered in `.claude-plugin/marketplace.json`. Note that a few **marketplace names differ from their directory names** (left column ≠ path).

| Marketplace name | Directory | What it does |
| --- | --- | --- |
| `prompt` | `plugins/prompt-library` | Prompt-template management: create, search, run, compose, orchestrate |
| `gitignore-guardian` | `plugins/gitignore-guardian` | `.gitignore` management + protective PreToolUse hooks |
| `dir-dr` | `plugins/dir-dr` | Directory Doctor — audit, map, and safely reorganize directory structure |
| `skills-hygiene` | `plugins/skills-hygiene` | Deduplicate, promote, audit, and generalize skill installations |
| `agent-teams-workforce` | `plugins/agent-teams-workforce` | Phase-gated SDLC agent workforce (primary, under active development) |
| `self-improving-agent` | `plugins/self-improving-agent` | Curate memory; promote learnings to rules and skills |
| `research-summarizer` | `plugins/research-summarizer` | Structured research summarization and briefs |
| `skills` | `plugins/skills` | Catch-all bundle of frequently-used personal skills |
| `my-editor` | `plugins/my-editor` | Personal editing and writing-style toolkit |
| `patent` | `plugins/patent` | Patent-prep toolkit: ideate → draft → triage (free public data only) |
| `obsidian` | `plugins/obsidian` | Obsidian toolkit: CLI, Bases, Canvas, Markdown, Defuddle |
| `cds` | `plugins/cds` | Customizable Design System — brand-neutral stylesheet/mock/component generator |
| `forge` | `plugins/forge` | Tooling for the FORGE framework — compose, revise, and review instructions an agent can execute without interpretation |
| `memre` | `plugins/memre` | Memory-and-deliverable hygiene: residue linter/fixer, PostToolUse hook, auto-memory audit, markdown-to-beads importer |
| `okf` | `plugins/okf` | Open Knowledge Format toolkit — author, convert, audit, enrich, and validate agent-readable knowledge bundles |

Each plugin documents itself. When you work inside a plugin, Claude Code auto-loads that plugin's `CLAUDE.md` — so this file does not link to them individually.

## Plugin Architecture

Each plugin lives in `plugins/<name>/`. A typical layout (not every plugin has every directory):

```text
plugins/<name>/
├── .claude-plugin/plugin.json   # Plugin manifest (name, version, description)
├── README.md                    # Human-facing docs
├── CLAUDE.md                    # Agent-facing docs (see "README.md vs CLAUDE.md")
├── commands/<command>.md        # Slash commands — each file = one /name:command
├── skills/<skill>/SKILL.md      # Auto-activating skills
├── agents/<agent>.md            # Subagent definitions (auto-discovered)
├── hooks/                       # Lifecycle hooks
└── scripts/                     # Automation (validation, generation, sync)
```

Commands are invoked as `/plugin-name:command`. Skills auto-activate when their `description` frontmatter semantically matches user intent.

## README.md vs CLAUDE.md

Every plugin should have both: a `README.md` (human-facing) and a `CLAUDE.md` (agent-facing). Claude Code auto-loads a plugin's `CLAUDE.md` when work happens inside that plugin — so root does not link to them individually.

**Do not duplicate content between them, and do not put CLAUDE.md content in a README or vice versa.** For information that serves both humans and agents, keep it in `README.md` and import it into the agent's context from `CLAUDE.md`:

```text
@README.md
```

`@` imports are whole-file only — there is no line-range or heading-fragment form. Paths resolve relative to the importing file, imports nest up to 4 levels deep, and an `@path` inside backticks or a code fence is left as literal text. To surface only part of a file, split that part into its own small file and import that.

Reference: <https://code.claude.com/docs/en/memory>

## Skill & Command Naming

When creating a new skill (or contemplating renaming an existing one), prioritize these guidelines:

- Use verb-noun naming (e.g. `compose-spec`, not `compose`).
- Avoid generic single-word names.
- Do not prefix a name with its own namespace/parent (no `forge-compose` inside `forge`).
- Exception: noun/role names are allowed for standing personas or agents (e.g. `pr-shepherd`).

## Frontmatter

**Commands (`commands/<name>.md`):**

```yaml
---
description: "One-line trigger description for Claude Code"
argument-hint: "[optional-arg]"
allowed-tools: [Read, Write, Bash, ...]
---
```

**Skills (`skills/<name>/SKILL.md`):**

```yaml
---
name: skill-name
description: >-
  Trigger conditions — this is what Claude Code matches against user intent
allowed-tools: [Read, Write, ...]
argument-hint: "[optional-arg]"
---
```

Skill authoring conventions: frontmatter present; description ≥ 50 chars; body ≥ 200 chars; no "last updated/modified" text; no mermaid code fence in a `SKILL.md` body.

## Build, Distribution & Workspaces

No build step and no npm scripts — plugins are plain Markdown/YAML. They are distributed via `.claude-plugin/marketplace.json`; install one by copying or symlinking its `plugins/<name>/` directory into your Claude config. When adding a new plugin, add an entry to `marketplace.json` pointing at its source directory.

`package.json` declares `plugins/*` as workspaces.

<!-- BEGIN AGENT TEAMS WORKFORCE: v:3.3.0 hash:b219767 -->

## Instructions for Agent Teams Workforce

This project uses an agentic framework names "Agent Teams Workforce" to...

These instructions will be loaded into the contect window at session start.
@/Users/msat1971/.claude/plugins/cache/mark-satterfield/agent-teams-workforce/3.3.0/AGENT-INSTRUCTIONS.md`

| Question about this:                                         | Find it here:                                                |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| SDLC pipelines, workflow scripts, agent taxonomy, teams, full roster. | [/Users/msat1971/.claude/plugins/cache/mark-satterfield/agent-teams-workforce/3.3.0/AGENT-TEAMS-WORKFORCE.md](/Users/msat1971/.claude/plugins/cache/mark-satterfield/agent-teams-workforce/3.3.0/AGENT-TEAMS-WORKFORCE.md) |
| Useful commands                                              | [/Users/msat1971/.claude/plugins/cache/mark-satterfield/agent-teams-workforce/3.3.0/commands](/Users/msat1971/.claude/plugins/cache/mark-satterfield/agent-teams-workforce/3.3.0/commands) |
<!-- END AGENT TEAMS WORKFORCE -->

---
> Source: [mark-satterfield/toolset](https://github.com/mark-satterfield/toolset) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
