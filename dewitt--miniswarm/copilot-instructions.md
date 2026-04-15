## miniswarm

> The swarm is built and running. Three autonomous agent runners (claude, gemini,

# Gemini CLI — Miniswarm Instructions

## Current project state (as of 2026-03-27)

The swarm is built and running. Three autonomous agent runners (claude, gemini,
codex) are connected to #swarm on localhost:6667. The runner daemon
(`scripts/runner.py`) bridges IRC with the agent CLI.

## What is this project?

Miniswarm is a multi-agent collaboration protocol. Multiple AI coding agents
and humans coordinate in real time over a local IRC server.

## IMPORTANT: Runner Environment

**DO NOT** manually connect to IRC or use `scripts/connect.sh`.
**DO NOT** run `tail` or `read.sh` to check for messages.
**DO NOT** read `AGENTS.md` or `design/SWARM-ARCHITECTURE.md` unless explicitly instructed to do so. The runner already provides all the protocol context you need in your system prompt.

You are being invoked by a stateless runner daemon (`scripts/runner.py`). The runner manages the persistent IRC connection for you.

When you are invoked:
1. The runner provides the latest IRC context via your system prompt.
2. You perform whatever tool calls are necessary to complete your task (editing files, reading code, etc.).
3. You output a concise, actionable text response using the prefixes like `STATUS`, `DONE`, `QUESTION`, or `HANDOFF`.
4. You finish and exit. The runner will capture your text response and post it back to the IRC channel on your behalf.

## Key rules

- Defer to human directives on IRC.
- Don't edit files another agent announced they're working on.
- Use feature branches, not main.
- Keep your final text response concise. Use `/tmp/swarm-share/` for large content if needed.
- Act autonomously to solve the problem without explicitly asking permission for every command.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dewitt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dewitt)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
