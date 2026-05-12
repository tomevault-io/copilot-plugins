## fsociety

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

fsociety is a multi-plugin marketplace for Claude Code offensive security plugins. Each plugin is a self-contained toolkit targeting a specific domain of penetration testing. The repo itself is NOT a Node.js application — there is no build step, no package.json, no test suite.

**Requirements**: Claude Code with MCP support, Kali Linux with Hexstrike MCP server, Node.js 18+.

## Repository Architecture

This is a **Claude Code plugin marketplace**. The root contains the marketplace registry and individual plugin directories:

- `.claude-plugin/marketplace.json` — Plugin registry listing all available plugins with their source directories
- Each plugin directory (e.g., `elliot/`) is a self-contained Claude Code plugin with its own `plugin.json`, `CLAUDE.md`, commands, skills, agents, scripts, and hooks

### Plugin Anatomy

Every plugin follows this structure:

```
plugin-name/
├── plugin.json           # Plugin definition (name, version, skills, agents)
├── CLAUDE.md             # Plugin-specific guidance for Claude Code
├── .gitignore            # Ignores runtime data files
├── commands/*.md         # Slash commands with YAML frontmatter
├── skills/*/SKILL.md     # Auto-activating skills with references/
├── agents/*.md           # Agent definitions with YAML frontmatter
├── scripts/*.js          # Node.js scripts (zero npm deps, stdlib only)
└── hooks/hooks.json      # Lifecycle hooks wiring scripts to events
```

### Marketplace Registration

`.claude-plugin/marketplace.json` is the index. To register a plugin, add an entry to the `plugins` array:

```json
{
  "name": "plugin-name",
  "description": "What it does.",
  "source": "./plugin-name/",
  "author": { "name": "author-name" }
}
```

### Plugin JSON Schema

Each plugin's `plugin.json`:

```json
{
  "name": "plugin-name",
  "version": "1.0.0",
  "description": "...",
  "author": { "name": "..." },
  "skills": ["./skills/", "./commands/"],
  "agents": ["./agents/agent-name.md"],
  "license": "MIT",
  "keywords": ["security", "pentest"]
}
```

## Current Plugins

| Plugin | Directory | Status | Domain |
|--------|-----------|--------|--------|
| **elliot** | `elliot/` | Available | Web & Application security — full offensive lifecycle |
| **romero** | `romero/` | Available | Reverse engineering — Windows binary analysis, decompilation, malware classification |
| **trenton** | `trenton/` | Available | Operational security — machine hardening, VPS security, anti-forensics, footprint elimination |
| **tyrell** | `tyrell/` | Available | Leak database hunting — exposed database discovery, data acquisition, cross-plugin pipeline to elliot |
| **fsociety** | `fsociety/` | Available | Engagement workspace setup — interactive wizard for targets, goals, scope, plugin selection, and cross-plugin orchestration |

See each plugin's `CLAUDE.md` for plugin-specific architecture, script CLI reference, data layer, and conventions.

Planned: **dom** (mobile/IoT).

## Engagement Setup

The `fsociety` plugin provides a `/setup` command to initialize engagement workspaces. This is the recommended starting point for any new operation.

### Quick Start

```bash
# Install the fsociety setup plugin
claude plugin install fsociety@fsociety

# Run the setup wizard
/setup my-operation
```

### What Setup Creates

| File | Purpose |
|------|---------|
| `engagement.json` | Central engagement config — targets, plugins, opsec, scope |
| `CLAUDE.md` | Tailored guidance with only active plugin commands/skills |
| `README.md` | Engagement overview and status |
| `scope.md` | Formal scope definition |
| `targets.jsonl` | Structured target list (append-only, SHA256-deduped) |
| `.gitignore` | Runtime data ignore patterns for all active plugins |

### OPSEC Profiles

| Profile | Speed | Anonymity | Use Case |
|---------|-------|-----------|----------|
| `surface` | Maximum | None | Lab/CTF |
| `standard` | Moderate | Basic | Authorized external tests |
| `paranoid` | Slow | Full (Tor/VPN) | Red team engagements |

## Conventions

- **Zero dependencies**: All scripts use only Node.js built-ins (`fs`, `path`, `crypto`, `child_process`)
- **Markdown-driven**: Commands, skills, and agents are defined in `.md` files with YAML frontmatter
- **Plugin isolation**: Each plugin is fully self-contained — no cross-plugin imports or shared scripts
- **Runtime data is gitignored**: Each plugin's `.gitignore` excludes generated data files (JSONL databases, reports, archives, session state)
- **Scripts reference `${CLAUDE_PLUGIN_ROOT}`**: Environment variable pointing to the plugin directory at runtime
- **JSONL for data**: Append-only JSONL files for findings, techniques, intel. JSON for session state
- **SHA256 deduplication**: All trackers deduplicate by hashing normalized key fields

## Command Frontmatter Schema

```yaml
---
description: What the command does
allowed-tools: Bash, Read, Write, Glob, Grep, Task, AskUserQuestion
argument-hint: <arguments>
---
```

## Agent Frontmatter Schema

```yaml
---
name: agent-name
description: |
  Trigger description with <example> blocks
color: cyan | red
tools: ["ToolSearch", "Bash", "Read", "Write", "Glob", "Grep"]
---
```

## Skill Structure

Each skill in `skills/<name>/`:
- `SKILL.md` — YAML frontmatter with `name`, `description` (trigger keywords), `version`
- `references/*.md` — Detailed methodology, tool parameters, and examples

## Adding a New Plugin

1. Create `plugin-name/` at the repo root
2. Add `plugin-name/plugin.json` following the schema above
3. Add `plugin-name/.gitignore` for runtime data files
4. Add `plugin-name/CLAUDE.md` with plugin-specific guidance
5. Create `commands/`, `skills/`, `agents/`, `scripts/`, `hooks/` subdirectories as needed
6. Register in `.claude-plugin/marketplace.json`

# context-mode — MANDATORY routing rules

You have context-mode MCP tools available. These rules are NOT optional — they protect your context window from flooding. A single unrouted command can dump 56 KB into context and waste the entire session.

## BLOCKED commands — do NOT attempt these

### curl / wget — BLOCKED
Any Bash command containing `curl` or `wget` is intercepted and replaced with an error message. Do NOT retry.
Instead use:
- `ctx_fetch_and_index(url, source)` to fetch and index web pages
- `ctx_execute(language: "javascript", code: "const r = await fetch(...)")` to run HTTP calls in sandbox

### Inline HTTP — BLOCKED
Any Bash command containing `fetch('http`, `requests.get(`, `requests.post(`, `http.get(`, or `http.request(` is intercepted and replaced with an error message. Do NOT retry with Bash.
Instead use:
- `ctx_execute(language, code)` to run HTTP calls in sandbox — only stdout enters context

### WebFetch — BLOCKED
WebFetch calls are denied entirely. The URL is extracted and you are told to use `ctx_fetch_and_index` instead.
Instead use:
- `ctx_fetch_and_index(url, source)` then `ctx_search(queries)` to query the indexed content

## REDIRECTED tools — use sandbox equivalents

### Bash (>20 lines output)
Bash is ONLY for: `git`, `mkdir`, `rm`, `mv`, `cd`, `ls`, `npm install`, `pip install`, and other short-output commands.
For everything else, use:
- `ctx_batch_execute(commands, queries)` — run multiple commands + search in ONE call
- `ctx_execute(language: "shell", code: "...")` — run in sandbox, only stdout enters context

### Read (for analysis)
If you are reading a file to **Edit** it → Read is correct (Edit needs content in context).
If you are reading to **analyze, explore, or summarize** → use `ctx_execute_file(path, language, code)` instead. Only your printed summary enters context. The raw file content stays in the sandbox.

### Grep (large results)
Grep results can flood context. Use `ctx_execute(language: "shell", code: "grep ...")` to run searches in sandbox. Only your printed summary enters context.

## Tool selection hierarchy

1. **GATHER**: `ctx_batch_execute(commands, queries)` — Primary tool. Runs all commands, auto-indexes output, returns search results. ONE call replaces 30+ individual calls.
2. **FOLLOW-UP**: `ctx_search(queries: ["q1", "q2", ...])` — Query indexed content. Pass ALL questions as array in ONE call.
3. **PROCESSING**: `ctx_execute(language, code)` | `ctx_execute_file(path, language, code)` — Sandbox execution. Only stdout enters context.
4. **WEB**: `ctx_fetch_and_index(url, source)` then `ctx_search(queries)` — Fetch, chunk, index, query. Raw HTML never enters context.
5. **INDEX**: `ctx_index(content, source)` — Store content in FTS5 knowledge base for later search.

## Subagent routing

When spawning subagents (Agent/Task tool), the routing block is automatically injected into their prompt. Bash-type subagents are upgraded to general-purpose so they have access to MCP tools. You do NOT need to manually instruct subagents about context-mode.

## Output constraints

- Keep responses under 500 words.
- Write artifacts (code, configs, PRDs) to FILES — never return them as inline text. Return only: file path + 1-line description.
- When indexing content, use descriptive source labels so others can `ctx_search(source: "label")` later.

## ctx commands

| Command | Action |
|---------|--------|
| `ctx stats` | Call the `ctx_stats` MCP tool and display the full output verbatim |
| `ctx doctor` | Call the `ctx_doctor` MCP tool, run the returned shell command, display as checklist |
| `ctx upgrade` | Call the `ctx_upgrade` MCP tool, run the returned shell command, display as checklist |

---
> Source: [ogrodev/fsociety](https://github.com/ogrodev/fsociety) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
