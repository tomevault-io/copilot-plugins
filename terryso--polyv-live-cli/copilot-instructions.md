## polyv-live-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **PolyV Live Monorepo** - a TypeScript/Node.js project containing:
- **SDK** (`polyv-live-api-sdk`): PolyV Live API SDK
- **CLI** (`polyv-live-cli`): Command-line interface for PolyV live streaming services

## Monorepo Structure

```
polyv-cli/
├── packages/
│   ├── sdk/                     # polyv-live-api-sdk
│   │   ├── src/
│   │   │   ├── services/        # API services (v3, v4)
│   │   │   ├── types/           # TypeScript type definitions
│   │   │   └── index.ts         # SDK entry point
│   │   └── package.json
│   └── cli/                     # polyv-live-cli
│       ├── src/
│       │   ├── commands/        # CLI command definitions
│       │   ├── handlers/        # Business logic
│       │   ├── config/          # Configuration management
│       │   └── index.ts         # CLI entry point
│       └── package.json
├── docs/                        # Project documentation
│   └── api/                     # API documentation
│       ├── coupon/              # Coupon API docs
│       ├── product/             # Product API docs
│       └── channel/             # Channel API docs
├── pnpm-workspace.yaml          # Workspace configuration
└── package.json                 # Root package.json
```

## Common Commands

```bash
# Install dependencies (from root)
pnpm install

# Build all packages
pnpm build

# Build specific package
pnpm --filter polyv-live-api-sdk build
pnpm --filter polyv-live-cli build

# Run unit tests
pnpm test:unit

# Run tests for specific package
pnpm --filter polyv-live-cli test:unit
```

## SDK Architecture

### Services

| Service | Location | Description |
|---------|----------|-------------|
| ChannelService | `packages/sdk/src/services/channel.service.ts` | Channel CRUD operations |
| V4PlatformService | `packages/sdk/src/services/v4/platform.service.ts` | Coupon management |
| AccountService | `packages/sdk/src/services/account.service.ts` | Account operations |

### Types

| Type File | Description |
|-----------|-------------|
| `types/channel.ts` | Channel and Product types |
| `types/v4-platform.ts` | Coupon types |
| `types/auth.ts` | Authentication types |

### Usage Example

```typescript
import { PolyVClient } from 'polyv-live-api-sdk';

const client = new PolyVClient({
  appId: 'your-app-id',
  appSecret: 'your-app-secret'
});

// Channel operations
const channels = await client.channel.listChannels({ pageNumber: 1, pageSize: 10 });

// Coupon operations (V4)
const couponId = await client.v4Platform.createCoupon({ ... });
const coupons = await client.v4Platform.searchCoupons({ pageNumber: 1 });

// Product operations
const products = await client.channel.getProductList({ channelId: 'xxx' });
```

## CLI Architecture

### Command Pattern

Commands are defined in `packages/cli/src/commands/*.commands.ts`:
- Use Commander.js framework
- Follow the pattern: `program.command('resource action').description('...').action(handler)`
- Handler files in `packages/cli/src/handlers/`

### Adding New Commands

1. Create command file: `src/commands/coupon.commands.ts`
2. Create handler file: `src/handlers/coupon.handler.ts`
3. Register in `src/index.ts`
4. Add types if needed: reference SDK types

### Authentication

CLI supports multiple auth sources (priority order):
1. Command-line parameters: `--appId`, `--appSecret`, `--userId`
2. Session account: `polyv-live-cli use <account-name>`
3. Environment variables: `POLYV_APP_ID`, `POLYV_APP_SECRET`
4. Default account: `polyv-live-cli account set-default <name>`
5. Global config: `polyv-live-cli config set`

## API Documentation

When implementing new features, always refer to the API docs:

| Feature | Doc Location |
|---------|--------------|
| Coupon | `docs/api/coupon/README.md` |
| Product | `docs/api/product/` |
| Channel | `docs/api/channel/` |

## Signature Mechanism

PolyV API uses MD5-based signature authentication:
1. Sort parameters by key name
2. Concatenate: `key1value1key2value2...`
3. Build: `appSecret + concatenated_params + appSecret`
4. Generate MD5 hash (uppercase)

## Development Guidelines

- Use `workspace:*` for internal package dependencies
- Follow existing command patterns in CLI
- Reference SDK types from `polyv-live-api-sdk`
- Run `pnpm test:unit` before committing
- Coverage target: 80%

## Package Manager

This project uses **pnpm** with workspace support:
- Use `pnpm` instead of `npm`
- Workspace protocol: `"polyv-live-api-sdk": "workspace:*"`
- Changesets for versioning: `pnpm changeset`

---
> Source: [terryso/polyv-live-cli](https://github.com/terryso/polyv-live-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
