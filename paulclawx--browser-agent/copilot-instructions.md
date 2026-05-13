## browser-agent

> browser-agent is a thin layer that connects agents to browsers via an editable CDP harness.

browser-agent is a thin layer that connects agents to browsers via an editable CDP harness.

# Code priorities
- Clarity
- Precision
- Low verbosity
- Versatility

# Overview
Core code lives in `src/browser_harness/`:
- `admin.py` — daemon lifecycle, diagnostics, updates, profile management
- `daemon.py` — the long-lived middleman process between the browser and the agent
- `helpers.py` — CDP wrapper and core browser primitives auto-imported into `-c` scripts
- `run.py` — the `browser-agent` CLI

`SKILL.md` tells agents how to use the harness and CLI.
`install.md` tells agents how to install it, attach a browser, and troubleshoot.

An agent operating the harness only edits inside `src/agent-workspace/`:
- `agent_helpers.py` — task-specific browser helpers the agent adds
- `docs/domain-skills/` — skills the agent writes and reads

# Contributing
Consider what is really needed. Prefer the smallest diff that fixes the bug.

---
> Source: [PaulClawX/browser-agent](https://github.com/PaulClawX/browser-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
