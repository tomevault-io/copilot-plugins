## skills

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repository contains custom extensions for Claude Code: slash commands, skills, and agent configurations.

## Architecture

- `.claude/commands/` - Slash commands as `.md` files. Filename (without extension) becomes the command name.
- `skills/` - Custom skills that extend Claude's capabilities with specialized knowledge and workflows.
- `agents/` - Agent configurations for specialized task handling.

## Adding Extensions

**Commands**: Create `<name>.md` in `.claude/commands/`. The file content becomes the prompt when `/name` is invoked.

**Skills**: Add skill definitions in `skills/` directory.

**Agents**: Add agent configurations in `agents/` directory.

---
> Source: [alexjx/skills](https://github.com/alexjx/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
