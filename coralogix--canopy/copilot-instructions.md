## canopy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Canopy is infrastructure for building agent-native applications with pluggable runtimes. The core abstraction is the **Workspace** — an isolated directory-based workspace that defines an agent's behavior, memory, and capabilities through `.md` files and runtime configuration.

**How it works:** Developers author a template folder (CLAUDE.md, skills, agents, tool configs). Canopy infra provisions a Workspace directory for every user from that template, handles triggers (scheduler, chat, webhooks), and spawns isolated runtime instances per user. The runtime returns results as stdout; Canopy captures and routes the output. Claude Code is the default runtime; any coding agent can be configured via `CANOPY_RUNTIME`.

**Philosophy:** Documents are database. Skills are functions. The LLM is the CPU. Code is only for I/O.

**Doc purpose:** These docs are an implementation spec — copy this folder into your project, point Claude at it, and it can build the entire Canopy platform for your agent.

**Status:** Documentation + templates + CLI (`cli/`). 90+ tests, CI via GitHub Actions, PyPI-ready packaging.

## Repository Structure

- `specs/01-architecture.md` through `specs/07-adoption-guide.md` — Framework specification docs
- `templates/` — Starter templates (CLAUDE.md.template, skill.md.template, tool-config.md.template, settings.json.template, workspace-structure.md, version.json)
- `examples/` — Example agent templates (`codebase-navigator`, `release-pilot`, `daily-standup`)
- `cli/` — Developer CLI for managing workspaces (create, run, connect, delete)

## Core Architecture

**Configurable Runtime:** Every Workspace is a directory. Canopy spawns a runtime process against that directory per trigger. The runtime handles context loading, skills, agents, MCP tools, memory, hooks, and sandboxing. No custom runtime code. Runtime is configurable via `CANOPY_RUNTIME` (default: `claude-code`) — see `specs/06-runtime.md`.

**3-Layer Workspace Model** (assembly order: L1 → L2 → L3):
1. **Layer 1 (Template)** — Developer-controlled, immutable at runtime. Agent personality, skills, agents, tool configs, settings.
2. **Layer 2 (User)** — User customizations, validated writes only. Preferences, custom instructions.
3. **Layer 3 (Agent)** — Agent-generated via runtime memory system. Memory, knowledge, learned skills. (When using Claude Code: auto-memory.)

Higher layers append but **cannot remove or override** Layer 1 safety rules.

**B2B extends to 4 layers:** Template → Org (admin-controlled) → User → Agent memory.

**Canopy Infra:** The thin infrastructure layer that provisions Workspace directories from templates, handles triggers, spawns runtime processes, captures stdout, and manages auth/billing.

## Writing Guidelines

- **Generic only** — Never reference specific commercial products or internal product names. Use generic examples like "Personal EA", "SRE Agent", "Customer Support Agent".
- **Stack-agnostic** — No specific tech stack prescribed. Describe what needs to happen, not how to implement it.
- **Security model** — 7 controls: template immutability, scoped agent writes, tool allowlist, sandboxed learned skills, validated user overrides, workspace isolation, token vault. All must be preserved in any changes.
- **Implementable spec** — Docs must be precise enough that Claude can implement the entire Canopy platform from them.
- **Template files** use `{{ placeholder }}` syntax with inline comments explaining what to fill in.
- **Workspace directories** (`workspaces/`) are gitignored — never commit user workspaces.
- **Runtime-agnostic language** — Use "the runtime" instead of "Claude Code" when describing general Canopy behavior. Qualify CC-specific details: "When using Claude Code: skills in `.claude/commands/`, agents in `.claude/agents/`, memory in `.claude/projects/*/memory/`, settings in `.claude/settings.json`."

---
> Source: [coralogix/canopy](https://github.com/coralogix/canopy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
