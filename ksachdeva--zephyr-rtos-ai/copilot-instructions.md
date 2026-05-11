## zephyr-rtos-ai

> This document provides context for AI coding assistants (Open Code, Claude Code, Gemini CLI, GitHub Copilot, etc.) to understand the `zephyr-rtos-ai` project and assist with development of features.

# AI Coding Assistant Context

This document provides context for AI coding assistants (Open Code, Claude Code, Gemini CLI, GitHub Copilot, etc.) to understand the `zephyr-rtos-ai` project and assist with development of features.

## Repository Structure

```
├── .gemini                                 # Gemini CLI related stuff (settings.json, commands, agents etc)
├── .agents                                 # Contains skills, commands etc for various different Coding agents that respect .agents folder
├── .gitattributes
├── .gitignore
├── .pre-commit-config.yaml
├── .python-version
├── .venv                                   # `uv` created `.venv`
├── .vscode
├── skills                            # The Agent Skills for Zephyr RTOS
│   ├── zephyr-devicetree
│   ├── zephyr-kconfig
│   ├── zephyr-kernel-datapassing
│   ├── zephyr-kernel-synchronization
│   └── zephyr-shell-commands
├── AGENTS.md                               # This file
├── internal                                # Skills, Agents, Extra stuff to assist with the development in this repository
│   ├── assets
│   └── zephyr-skill-creator                # A skill that guides with the creation of other skills
├── LICENSE
├── poe.toml
├── pyproject.toml
├── README.md
├── src                                     # A python project that would contain useful utils
│   └── zephyr_rtos_ai
├── uv.lock
└── zephyr-ws                               # The Zephyr workspace, mostly exists for testing & agents to access documentation
    ├── .west
    └── west.yml
```

## Project Overview

The primary objective of this repository is to create reusable AI agents, skills, tools etc for Zephyr RTOS based firmware development.

You have access to a meta skill called `zephyr-skill-creator` that acts as the guide on how to create
various other skills.

## Development Setup

This is a `uv` project which means that you MUST run any `python` script/app using `uv run` so that
the proper `virtualenv` is used.

`poe.toml` file contains reusable tasks that generally wrap other tools.

## Prohibited

You are *NOT* allowed to make changes to following development setup files:

- poe.toml
- pyproject.toml
- uv.lock

You are *NOT* allowed to run following commands:

- `uv run poe west-setup`

---
> Source: [ksachdeva/zephyr-rtos-ai](https://github.com/ksachdeva/zephyr-rtos-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
