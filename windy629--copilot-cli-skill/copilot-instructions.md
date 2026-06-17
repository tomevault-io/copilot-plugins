## copilot-cli-skill

> Provide an OpenClaw skill that wraps GitHub Copilot CLI (copilot) commands so agents can run Copilot CLI features (suggest, chat, autopilot, agents) programmatically and receive structured results.

# Copilot CLI Skill for OpenClaw

Purpose
-------
Provide an OpenClaw skill that wraps GitHub Copilot CLI (copilot) commands so agents can run Copilot CLI features (suggest, chat, autopilot, agents) programmatically and receive structured results.

Files created
-------------
- scripts/run-copilot.sh — wrapper to call `copilot` with args and return JSON (exit/stdout/stderr).
- scripts/run-copilot-programmatic.py — Python helper for programmatic calls with argument array and timeout support.
- SKILL.md — this file with usage and security notes.
- examples/README.md — usage examples.

Requirements
------------
- `copilot` CLI must be installed and on PATH. See https://docs.github.com/copilot/how-tos/copilot-cli for installation and auth.
- Authenticated user with Copilot access (copilot auth login or environment variables). For programmatic automation prefer using a machine/service principal per GitHub docs.

Security
--------
- Commands run with OpenClaw process privileges. Do not pass untrusted input unsanitized. Use argument arrays, not shell interpolation.
- Limit which agents/skills can call this skill via OpenClaw permissions.

Usage
-----
- From an agent: call /home/km/.openclaw/workspace/skills/copilot-cli-skill/scripts/run-copilot.sh <args>
- Or for structured calls with timeout: python3 scripts/run-copilot-programmatic.py --timeout 30 -- args...

Examples
--------
- ./scripts/run-copilot.sh chat "Explain this code" --json
- python3 scripts/run-copilot-programmatic.py --timeout 20 -- suggest --file path/to/file --json

Next steps
----------
I can:
A) Install `copilot` CLI on this host and run an auth check (requires your approval).
B) Add a small OpenClaw skill entry (register) so agents can call this skill by name instead of raw paths.
C) Implement additional helpers (pre-built prompts, templates, output parsers for JSON).

---
> Source: [windy629/copilot-cli-skill](https://github.com/windy629/copilot-cli-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
