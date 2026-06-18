## agent-comms

> A simple markdown-based communication channel for any coding agent and humans. Everyone reads and writes to the same conversation log. Humans can participate, review what agents are saying, and change the rules. Use when you need task handoffs, reviews, status checks, or a shared message trail across agent boundaries.


# Agent Comms

## Overview

Use this skill to initialize and operate a single shared communication channel for multiple agents and humans in a repository.

If the repo does not already contain the comms files, bootstrap them first with `scripts/setup.sh`. After that, use the normal workflow: read the rules first, then check the active log, then reply or append at the bottom.

## Quick Start

Install:

```bash
npx skills add Abbiirr/agent-comms
```

For Claude Code with non-interactive install:

```bash
npx skills add Abbiirr/agent-comms --agent claude-code --yes --copy
```

After install, invoke the skill normally.

- Codex: `Use $agent-comms to bootstrap this repository.`
- Claude Code: `/agent-comms bootstrap this repository`

On first invocation, if the repo is missing the comms files, this skill should bootstrap them automatically before doing anything else.

Local development install flow:

```bash
npx skills add /path/to/agent-comms --agent codex claude-code --yes --copy
```

Fallback manual bootstrap flow:

```bash
bash .agents/skills/agent-comms/scripts/setup.sh
```

What bootstrap does:

- bootstraps `AGENT_COMMUNICATION_RULES.md`
- bootstraps `AGENTS_CONVERSATION.MD`
- ensures `docs/communication/old/` exists
- injects a standard comms section into `AGENTS.md` by default

After setup:

1. Read `AGENT_COMMUNICATION_RULES.md`
2. Read `AGENTS_CONVERSATION.MD`
3. Report:
   - active entries
   - messages requiring a response
   - threads ready for archival
4. Before changing code or docs, append a pre-task intent entry
5. Reply or append new entries at the bottom

## Bootstrap Workflow

Use `scripts/setup.sh` as the public entrypoint. It wraps `init_agent_comms.py` and gives a cleaner CLI.

Use bootstrap when the repo does not already have:

- `AGENT_COMMUNICATION_RULES.md`
- `AGENTS_CONVERSATION.MD`
- `docs/communication/old/`

Default commands:

```bash
# Codex
bash .agents/skills/agent-comms/scripts/setup.sh

# Claude Code
bash .claude/skills/agent-comms/scripts/setup.sh
```

Useful flags:

- `--root <path>` to target a different repo root
- `--no-inject-agents-md` to avoid modifying `AGENTS.md`
- `--force` overwrites the template files
- `--inject-agents-md` explicitly enables `AGENTS.md` injection
- `--python <bin>` chooses the Python interpreter
- `--rules-file`, `--conversation-file`, and `--archive-dir` let you override the defaults

Bootstrap should be conservative:

- do not overwrite existing files unless `--force` is set
- do not read archives by default
- do not auto-resolve threads

## First Invocation Behavior

When this skill is invoked in a repo that does not yet contain the comms files, bootstrap the repo first, then continue with the requested comms work.

Missing bootstrap files means one or more of these is absent:

- `AGENT_COMMUNICATION_RULES.md`
- `AGENTS_CONVERSATION.MD`
- `docs/communication/old/`

Use the first valid setup script path that exists:

1. `.agents/skills/agent-comms/scripts/setup.sh`
2. `.claude/skills/agent-comms/scripts/setup.sh`
3. `${CLAUDE_SKILL_DIR}/scripts/setup.sh` if that environment variable exists

Run the bootstrap script from repo root:

```bash
bash .agents/skills/agent-comms/scripts/setup.sh
```

or, for Claude Code if needed:

```bash
bash .claude/skills/agent-comms/scripts/setup.sh
```

After bootstrap:

1. tell the user what files were created
2. read `AGENT_COMMUNICATION_RULES.md`
3. read `AGENTS_CONVERSATION.MD`
4. continue with the requested comms task

If the files already exist, do not re-run bootstrap unless the user explicitly asks to reinitialize or overwrite with `--force`.

## Operating Workflow

### 1. Identify the agent

Use your actual environment identity, such as Codex, Claude, OpenCode, or User.

### 2. Read the rules first

Always read `AGENT_COMMUNICATION_RULES.md` before `AGENTS_CONVERSATION.MD`.

### 3. Check for pending messages

Look for:

- `Directed to: <your name>`
- `Replying to: <your name>`
- open questions or task handoffs without a response
- resolved threads that the original author can archive

Report:

- number of active entries
- messages requiring a response
- threads ready for archival

### 4. Take action

Before any code or doc changes:

- append a pre-task intent entry

When replying:

- use the correct message type
- append to the bottom of `AGENTS_CONVERSATION.MD`
- preserve the identity header format from the rules file

When archiving:

- only the original author archives, unless the user explicitly overrides
- move the full resolved thread to `docs/communication/old/`
- never delete archived files

## Guardrails

- Never read `docs/communication/old/` unless the user explicitly asks, or an active thread points to a specific archive file.
- Never self-resolve another agent's entry.
- Never let a bootstrap command silently overwrite an existing comms setup without `--force`.
- Keep `AGENTS_CONVERSATION.MD` lean and append-only during normal operation.

## Resources

- `scripts/setup.sh` — production-facing bootstrap entrypoint
- `scripts/init_agent_comms.py` — bootstrap a repo-local comms setup
- `assets/AGENT_COMMUNICATION_RULES.md` — default rules template
- `assets/AGENTS_CONVERSATION.MD` — default active-log template
- `assets/AGENTS_MD_SNIPPET.md` — optional snippet to add to a repo's `AGENTS.md`

---
> Source: [Abbiirr/agent-comms](https://github.com/Abbiirr/agent-comms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
