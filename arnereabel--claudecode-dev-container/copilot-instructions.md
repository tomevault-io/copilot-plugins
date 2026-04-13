## claudecode-dev-container

> This file provides context and instructions for Claude Code when working in this repository.

# CLAUDE.md - Instructions for Claude Code

This file provides context and instructions for Claude Code when working in this repository.

## Project Overview

This is a secure Claude Code development container running on a VPS. The environment includes:
- **Runtime**: Node.js 20
- **Shell**: ZSH with Powerlevel10k
- **Editor**: nano (default), vim available
- **Package Manager**: npm

## Project: FPX

The `/fpx` directory is the main project workspace. (Project details to be added as development progresses.)

## Development Guidelines

### Code Style
- Use Prettier for formatting (auto-format on save enabled)
- ESLint for linting JavaScript/TypeScript
- Follow existing patterns in the codebase

### Git Workflow
- Commit early and often with descriptive messages
- Branch naming: `feature/`, `fix/`, `chore/`
- Always pull before pushing

### Build & Test Commands
```bash
# Add project-specific commands here as they're established
npm install
npm run dev
npm test
```

## Security Context

This container runs with a restricted firewall. Network access is limited to:
- npm registry
- GitHub
- Claude API (api.anthropic.com)
- VS Code marketplace

**You cannot access arbitrary URLs.** If you need to add a domain to the whitelist, edit `/usr/local/bin/init-firewall.sh`.

## Container Info

- **Workspace**: `/workspace` (mounted from host)
- **Config**: `/home/node/.claude` (persisted via volume)
- **History**: `/commandhistory/.bash_history` (persisted)

## Useful Commands

| Command | Description |
|---------|-------------|
| `/init` | Create project CLAUDE.md |
| `/stats` | View token usage |
| `/compact` | Clear message history |
| `/help` | View all commands |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arnereabel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/arnereabel)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
