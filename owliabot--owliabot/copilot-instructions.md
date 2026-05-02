## owliabot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OwliaBot is a self-hosted, crypto-native AI agent designed for cryptocurrency users. It prioritizes security (local-first, minimal dependencies) and supports Telegram + Discord channels.

**Status**: MVP functional. Telegram/Discord channels working, 7 built-in tools, agentic loop operational.

## Commands

```bash
npm install          # Install dependencies
npm run dev          # Run with hot reload (tsx watch)
npm run build        # Compile TypeScript
npm run start        # Run compiled version
npm run typecheck    # Type check without emitting
npm run lint         # ESLint
```

## Architecture

### Core Layers
1. **Entry Layer**: CLI (commander) → Config (YAML + Zod) → Gateway
2. **Channel Layer**: Telegram (grammy) + Discord (discord.js) via unified `ChannelPlugin` interface
3. **Agent Runtime**: System prompt builder → LLM runner (multi-provider with failover) → Tool executor
4. **Signer Layer**: 3-tier key security model

### 3-Tier Key Security Model
| Tier | Type | Use Case |
|------|------|----------|
| 1 | Companion App (iOS/Android) | User-confirmed transactions; key never leaves app |
| 2 | Session Key (local, disposable) | Small automated ops (gas, claims); amount-limited |
| 3 | Smart Contract Wallet (ERC-4337) | Large automated ops; granular permissions; revocable |

### Directory Structure
```
src/
├── entry.ts              # CLI entry point (commander)
├── config/schema.ts      # Zod config schema
├── channels/interface.ts # ChannelPlugin interface
├── agent/
│   ├── runner.ts         # LLM runner with failover
│   ├── session.ts        # Session management
│   └── tools/interface.ts
├── signer/interface.ts   # 3-tier signing interface
├── notifications/service.ts
└── utils/logger.ts
```

### Workspace Structure (Runtime Config)
```
workspace/
├── SOUL.md         # Personality (tone, boundaries)
├── IDENTITY.md     # Bot identity (name "Owlia", emoji 🦉)
├── USER.md         # User preferences, timezone
├── HEARTBEAT.md    # Recurring checklist tasks
├── MEMORY.md       # Long-term decisions
└── memory/         # Episodic memory (diary/, weekly/, archive/)
```

## Built-in Tools

| Tool | Level | Description |
|------|-------|-------------|
| `echo` | read | Test tool, echoes input |
| `help` | read | List available tools |
| `clear_session` | write | Clear conversation history |
| `memory_search` | read | Search through memory files |
| `memory_get` | read | Read specific lines from a file |
| `list_files` | read | List directory contents |
| `edit_file` | write | Edit file with precise text replacement |

## Key Interfaces

**ChannelPlugin**: `{ id, start(), stop(), onMessage(), send(), capabilities }`

**ToolDefinition**: `{ name, description, parameters, security: { level, confirmRequired, maxValue, allowedContracts }, execute() }`

**SignerInterface**: `{ getAddress(), signMessage(), signTransaction(), sendTransaction(), tier, canAutoSign, maxAutoSignValue }`

## Technology Stack

- **Language**: TypeScript (ESM)
- **CLI**: commander
- **Config**: YAML + Zod validation
- **Logging**: tslog
- **HTTP**: undici
- **Cron**: croner
- **Channels**: grammy (Telegram), discord.js (Discord)
- **AI**: Direct HTTP or @anthropic-ai/sdk, openai (optional)
- **Crypto**: viem or ethers (as-needed)

## Design Constraints

- Target < 30 direct dependencies, < 150 total packages
- No native modules unless necessary
- No browser automation (no Playwright)
- Private keys never enter bot process
- Only 2 channels (TG + Discord) for security/auditability

---
> Source: [owliabot/owliabot](https://github.com/owliabot/owliabot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
