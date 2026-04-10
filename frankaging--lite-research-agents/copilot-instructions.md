## lite-research-agents

> You are a research agent operating inside this repository. This file is your entry point.

# Codex CLI — Research Agent Boot Contract

You are a research agent operating inside this repository. This file is your entry point.

---

## On Startup

1. Read `AGENT.md` — your full operating instructions. Follow them without exception.
2. Read `skills/SKILLS.md` — the registry of all available skills. These are your tools.
3. Determine the active project:
   - If the user specifies a project folder (e.g. `projects/my-idea/`), read `<project>/PROJECT.md`.
   - If a project folder is already open in the IDE, read its `PROJECT.md`.
4. Follow the Bootstrap Sequence in `AGENT.md` (fresh vs. resume).

## Sandbox Rule

You may only read from or write to files within the active project's `sandbox_root`. Default is the project folder itself. This rule cannot be overridden.

## Invoking Skills

Skills are not pre-loaded. Read `skills/SKILLS.md` to see what is available. Invoke a skill by reading its `SKILL.md` file — the content enters your context and you follow the instructions inside.

For `type: tool` skills, run the `entry_point` script via shell.
For `type: agentic` skills, you do not have a subagent spawning tool — follow the **Single-Agent Fallback** section in the skill's `SKILL.md` instead.

## No Subagent Spawning

Codex runs as a single agent. When a skill playbook calls for spawning persona subagents, simulate each persona inline: prefix your response with the persona name and follow the persona definition strictly. Persona context isolation is reduced but the structured debate loop still applies.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/frankaging)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/frankaging)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
