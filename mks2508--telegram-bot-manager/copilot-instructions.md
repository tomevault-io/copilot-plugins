## telegram-bot-manager

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**@mks2508/telegram-bot-manager** is a TypeScript library AND CLI for automating Telegram bot management via BotFather. It uses GramJS (MTProto) to interact with Telegram's protocol directly, enabling automation of tasks not possible with the Bot API alone.

### Dual Purpose

1. **CLI** (via npx): `npx @mks2508/telegram-bot-manager bootstrap`
2. **Library** (via import): `import { BotFatherManager } from '@mks2508/telegram-bot-manager'`

### Core Capabilities

1. **BotFather Automation** (`BotFatherManager`): Create bots, set commands, configure descriptions/about text, retrieve tokens
2. **Group/Forum Management** (`GroupManager`): Create supergroups, convert to forums, manage admin rights
3. **Topic Management** (`TopicManager`): Create/delete forum topics, manage topic configurations
4. **Environment Management** (`EnvManager`): Multi-bot configuration with .env file generation
5. **Interactive Bootstrap** (`BootstrapState`): Guided setup workflow for complete bot environment configuration

---

## Package Structure

```
telegram-bot-manager/
├── core/packages/main/          # Main & only package: @mks2508/telegram-bot-manager
│   ├── src/
│   │   ├── index.ts             # Library exports
│   │   ├── client.ts            # BootstrapClient (GramJS wrapper)
│   │   ├── bot-father/          # BotFather automation
│   │   │   ├── index.ts         # BotFatherManager
│   │   │   ├── message-handler.ts
│   │   │   ├── button-handler.ts
│   │   │   ├── parsers.ts
│   │   │   └── logger.ts
│   │   ├── group-manager.ts     # GroupManager
│   │   ├── topic-manager.ts     # TopicManager
│   │   ├── env-manager.ts       # EnvManager
│   │   ├── bootstrap-state.ts   # BootstrapState
│   │   ├── config/              # JSON configs (commands, topics)
│   │   ├── types/               # TypeScript types
│   │   └── cli/                 # CLI code (bundled separately)
│   │       ├── index.ts         # CLI entry point
│   │       ├── commands/        # bootstrap, bot, configure, topics
│   │       └── utils/           # CLI utilities (logger, spinner, helpers)
│   ├── dist/
│   │   ├── index.js             # Library (ESM)
│   │   ├── index.d.ts           # Type declarations
│   │   └── bin/cli.js           # Bundled CLI (works with Node & Bun)
│   ├── package.json
│   ├── tsconfig.build.json
│   └── README.md
├── .changeset/                   # Changesets config
└── package.json                  # Root (monorepo config)
```

---

## Commands

### Build & Development

```bash
# Root level
bun install                       # Install dependencies
bun run build                     # Build library + CLI
bun run typecheck                 # Type check (tsgo)
bun run lint                      # Lint (oxlint)

# Package level
cd core/packages/main
bun run build                     # tsgo + bun build CLI
bun run build:lib                 # Library only (tsgo)
bun run build:cli                 # CLI bundle only
```

### CLI Usage

```bash
# Via npx (after publishing)
npx @mks2508/telegram-bot-manager bootstrap
npx @mks2508/telegram-bot-manager bot list
npx @mks2508/telegram-bot-manager configure commands mybot

# Direct execution (development)
node dist/bin/cli.js --help
bun dist/bin/cli.js bootstrap
```

### Publishing

```bash
bunx changeset                    # Create changeset
bunx changeset version            # Apply versions
npm publish --access public       # Publish to npm
```

---

## Build Configuration

### package.json (key fields)

```json
{
  "name": "@mks2508/telegram-bot-manager",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "bin": {
    "telegram-bot-manager": "./dist/bin/cli.js",
    "tbm": "./dist/bin/cli.js"
  },
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  }
}
```

### Build Scripts

- **build:lib**: `tsgo -p tsconfig.build.json` - Compiles library + types
- **build:cli**: `bun build ./src/cli/index.ts --outfile ./dist/bin/cli.js --target node --minify` - Bundles CLI for Node.js

The CLI is bundled separately because:
1. Includes all dependencies (no ESM resolution issues)
2. Works with both Node.js and Bun
3. Single file (~1.6MB) for easy distribution

---

## Core Architecture

### Key Classes

**`BootstrapClient`** (`client.ts`)
- Wraps GramJS `TelegramClient` with session management
- Handles user authorization (phone, code, 2FA)
- Session persistence to `~/.mks-telegram-bot/session.txt`

**`BotFatherManager`** (`bot-father/index.ts`)
- Automates @BotFather via private messages
- Key methods: `createBot()`, `listBots()`, `getAllBotsWithTokens()`, `setCommands()`

**`GroupManager`** (`group-manager.ts`)
- Creates supergroups via `channels.CreateChannel`
- Converts to forums, promotes bots to admin

**`TopicManager`** (`topic-manager.ts`)
- Creates forum topics via Bot API
- Default topics: General, Control, Logs

**`EnvManager`** (`env-manager.ts`)
- Multi-bot `.env` file management
- Structure: `core/.envs/{botUsername}/{environment}.env`

### BotFather Automation Pattern

```typescript
async exampleOperation() {
  this.messageHandler.setupListener()
  await this.messageHandler.sleep(500)

  await this.messageHandler.sendMessage('/command')
  await this.messageHandler.sleep(2000)

  const response = await this.messageHandler.waitForResponse(10000)
  // Parse response, click buttons, send replies...

  this.messageHandler.removeListener()
  return { success: true }
}
```

---

## CLI Architecture

The CLI (`src/cli/`) is structured as:

- **index.ts**: Commander.js entry point with shebang
- **commands/**: Individual command handlers
  - `bootstrap.ts`: Complete bot setup wizard
  - `bot.ts`: Bot management (list, use, info, delete, migrate)
  - `configure.ts`: BotFather configuration (commands, description, about, name)
  - `topics.ts`: Forum topic creation
- **utils/**: CLI-specific utilities
  - `logger.ts`: better-logger wrapper with spinner support
  - `spinner-writer.ts`: Custom ora integration
  - `helpers.ts`: API credentials, env parsing
  - `names.ts`: Random bot name generators

### CLI Imports

CLI commands import from relative paths (not package index):
```typescript
// In src/cli/commands/bootstrap.ts
import { BootstrapClient } from '../../client.js'
import { BotFatherManager } from '../../bot-father/index.js'
```

---

## Development Guidelines

### Logging

```typescript
// Library code
import logger, { component } from '@mks2508/better-logger'
const log = component('BotFather')
log.info('Message')

// CLI code
import { cliLogger } from '../utils/logger.js'
cliLogger.success('Done!')
```

### Type Conventions

- Interfaces: `IOptions`, `IResult` (with I prefix)
- Types: `Environment`, `ErrorCode` (no prefix)

### JSDoc

All exports should have JSDoc with `@param`, `@returns`, `@example`.

---

## Troubleshooting

### Common Issues

**"No response from BotFather"**
- Check GramJS connection
- Increase timeout in `waitForResponse()`
- Try `/cancel` in Telegram to reset BotFather state

**"CLI not working with Node.js"**
- Ensure CLI is built with `--target node`
- Check shebang is `#!/usr/bin/env node`

**"Module resolution errors"**
- CLI uses direct imports, not package index
- Library exports from `src/index.ts`

### GramJS Notes

- Uses `@ts-ignore` for incomplete type definitions
- BigInteger handling with `big-integer` package
- Session cached at `~/.mks-telegram-bot/session.txt`

---

## License

MIT

---
> Source: [MKS2508/telegram-bot-manager](https://github.com/MKS2508/telegram-bot-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
