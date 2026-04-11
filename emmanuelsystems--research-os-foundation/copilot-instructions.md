## research-os-foundation

> This repository serves as a **foundational operating system** for research and development projects. It is designed to be read and executed by both humans and AI agents.

# Agent Entrypoint

## Purpose
This repository serves as a **foundational operating system** for research and development projects. It is designed to be read and executed by both humans and AI agents.

## Core Structure
The repository is organized into strict semantic sections. Agents should respect this hierarchy when reading or writing.

*   **`S00-foundation/`**: The "Kernel". Core rules, glossary, and purpose. **Read-only** for most agents.
*   **`S10-primitives/`**: The "Type System". Definitions of Agents, Skills, and other structural entities.
*   **`S20-workflows/`**: The "Procedures". Step-by-step guides for executing complex tasks. **Execute these**.
*   **`S30-examples/`**: The "Training Data". Golden samples of what good outputs look like. **Emulate these**.
*   **`S90-assets/`**: The "Filesystem". Raw assets, media, and supporting files.

## Agent Directives
1.  **Skills First**: Before attempting a complex task, check `.codex/skills/` or `S10-primitives/` for a defined skill.
2.  **Workflow Compliance**: If a task matches a workflow in `S20-workflows/`, you **must** follow it step-by-step.
3.  **Artifact Generation**: All work must result in a saved artifact. Do not leave results in chat only.
4.  **Context Loading**: When starting a new task, look for the `AGENTS.md` (this file) and the project's `README.md` to orient yourself.

## Available Skills
*   *Pending indexing of .codex/skills/*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/emmanuelsystems)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/emmanuelsystems)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
