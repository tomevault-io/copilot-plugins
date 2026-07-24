## cowork-skills

> CoWork Skills is a CLI tool (`cowork` / `co`) for managing Claude Code skills:

# CoWork Skills - Claude Instructions

## Overview

CoWork Skills is a CLI tool (`cowork` / `co`) for managing Claude Code skills:
- Install skills from GitHub repositories
- Generate skills from source code
- Search for skill repositories
- Manage Claude Code marketplace plugins

## CLI Tool

Use the `cowork` CLI to manage skills:

```bash
# Build and install CLI
cd cli && cargo install --path .

# Initialize built-in skills
cowork init

# Install from GitHub repository
cowork install user/repo

# Install specific skills
cowork install user/repo -s skill1 -s skill2

# Install to specific agents
cowork install user/repo -a claude-code -a cursor

# List installed repositories
cowork install --list

# Uninstall a repository
cowork install --uninstall repo-name

# Generate skills from source code
cowork generate user/repo --lang rust

# Search for skill repositories
cowork search agent-skill --topic

# List all skills
cowork list

# Check status
cowork status
cowork doctor
```

## Commands

| Command | Description |
|---------|-------------|
| `cowork init` | Install built-in skills to ~/.claude/skills/ |
| `cowork install` | Install skills from GitHub or local path |
| `cowork generate` | Generate skills from a GitHub repository |
| `cowork search` | Search GitHub for skill repositories |
| `cowork plugins` | Manage Claude Code marketplace plugins |
| `cowork list` | List all available skills |
| `cowork status` | Show current configuration |
| `cowork doctor` | Check for configuration issues |
| `cowork config` | Manage project-level skill configuration |
| `cowork test` | Generate trigger tests for skills |

## Storage Locations

| Location | Purpose |
|----------|---------|
| `~/.cowork/repos/` | Cloned GitHub repositories |
| `~/.claude/skills/` | Global skills directory |
| `./skills/` | Project-local skills |

## Global Skills (~/.claude/skills/)

Cross-project tools automatically available:

| Skill | When to Use |
|-------|-------------|
| `memory-filesystem` | remember, recall, reflect, memory |
| `best-skill-creator` | create skill, skill template |
| `writing-assistant` | writing, grammar, polish text |

## Default Project Settings

When creating Rust projects, use:

```toml
[package]
edition = "2024"
rust-version = "1.85"

[lints.clippy]
all = "warn"
pedantic = "warn"
```

---
> Source: [ZhangHanDong/cowork-skills](https://github.com/ZhangHanDong/cowork-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
