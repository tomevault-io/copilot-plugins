## home-pulse-watcher

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HomePulse Watcher is a DIY power outage monitoring system. ESP32-C3/ESP32-C6 devices monitor power status and send HMAC-signed REST requests to this NestJS backend, which stores events and notifies users via Telegram.

**MVP Stage**: This project is in early development. No legacy concerns - database can be recreated from scratch as needed.

> **AI Assistant Shared Mental Model**: [`PROJECT_CONTEXT.md`](PROJECT_CONTEXT.md) — load this for a concise overview of architecture, domain rules (ADC sensing, HMAC/AES, debounce), incident history, and doc map.

## Build & Development Commands

```bash
# Start development server
npx nx serve api

# Build for production
npx nx build api

# Run unit tests
npx nx test api

# Run single test file
npx nx test api --testFile=app.service.spec.ts

# Run E2E tests
npx nx e2e api-e2e

# Lint
npx nx lint api

# Type check
npx nx typecheck api

# Build Docker image
npx nx docker:build api

# Database commands
npx prisma generate
npx prisma migrate dev --name <migration_name>
npx prisma migrate deploy  # production
```

## Architecture

Full architectural overview: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)

### Core Layer Exports

```typescript
// Entities
(User, Device, UserDevice, PowerEvent);

// Enums
(PowerStatus((OFF = 0), (ON = 1)), DeviceRole(OWNER, EDITOR, VIEWER));

// Repository Interfaces
(IUserRepository, IDeviceRepository, IUserDeviceRepository, IPowerEventRepository);
```

### Infrastructure Layer (Pure TypeScript)

```typescript
// Repositories - plain classes, constructor injection of PrismaClient
const userRepo = new PrismaUserRepository(prismaClient);
const device = await userRepo.findByTelegramId(BigInt(123));

// Factory for PrismaClient singleton
(getPrismaClient(), disconnectPrisma());
```

### Telegram Bot Integration

- **Library**: Telegraf (TypeScript-native Telegram bot framework)
- **Module**: `apps/api/src/modules/telegram/`
- **Interaction model**: Button-driven; `/start` is the only slash command (user registration)
  - **Reply Keyboard**: Status, Devices, Settings, Help (persistent bottom menu)
  - **Inline Buttons**: Check Status, View History (in power event notifications)
  - **Settings**: Stateless inline keyboard menu for locale and timezone selection
- **Parse mode**: MarkdownV2 for all bot messages (`escape-markdown.ts` utility, `keyboard.builder.ts` for keyboard construction)
- **Notifications**: Event-driven via `@OnEvent(POWER_STATUS_CHANGED_EVENT)`
- **Authentication**: User verification via `telegramId` lookup before protected commands
- **Environment**: `TELEGRAM_BOT_TOKEN` (required), `TELEGRAM_ADMIN_CHAT_ID` (optional)
- **i18n**: `TranslationService` in `telegram/i18n/` — default locale `uk`, default timezone `Europe/Kyiv`. All user-facing strings are translated (Ukrainian + English). Date/time formatting uses per-user timezone via `Intl.DateTimeFormat`.

Bot follows the adapter pattern - handlers call existing Application Services, keeping business logic transport-agnostic.

### ESP32 Firmware

- **Location**: `firmware/` directory (ESP32-C3 and ESP32-C6 variants)
- **Build Tool**: PlatformIO with Arduino framework
- **Features**: WiFi, NTP time sync, HMAC-SHA256 signing, GPIO power detection
- **Configuration**: `config.h` (hardware), `secrets.h` (credentials - not tracked)
- **Documentation**: `docs/admin-guide.md`, `firmware/docs/FLASHING_GUIDE.md`

## Tech Stack

- **Monorepo**: Nx 22.4 with `@home-pulse-watcher/` prefix
- **Framework**: NestJS 11
- **Bundler**: Webpack via `@nx/webpack` (serverless bundling — all deps bundled except Prisma)
- **ORM**: Prisma 7.3 with PostgreSQL
- **Validation**: LIVR (custom rules use camelCase: `macAddress`, `hmacFormat`, `powerStatus`, `telegramId`)
- **Testing**: Jest 30 with SWC compiler
- **CLI**: nest-commander for admin tasks
- **Deployment**: Google Cloud Run (Docker multi-stage build)

## Database Models

- **User**: Telegram users (telegramId unique, locale default 'uk', timezone default 'Europe/Kyiv')
- **Device**: ESP32 devices (macAddress unique, encryptedSecret for HMAC verification)
- **UserDevice**: Many-to-many with role (VIEWER default)
- **PowerEvent**: Status changes (1=on, 0=off) with optional duration

## Build & Bundling

Webpack bundles **all** `node_modules` into `main.js` and `cli.js`, eliminating runtime dependency resolution issues. Only Prisma-related packages are kept external (they require native WASM binaries and generated code at runtime).

- **Config**: `apps/api/webpack.config.js`
- **External packages**: `@prisma/client`, `@prisma/adapter-pg`, `.prisma/client`, `pg`
- **Minimal package.json**: `apps/api/src/assets/package.json` — copied to `dist/`, lists only external deps for Docker `npm install`
- **NestJS lazy imports**: `@nestjs/microservices` and `@nestjs/websockets` are ignored via `IgnorePlugin` (optional peer deps, not installed)

## Coding Standards

- **Strict TypeScript**: No `any`, strict null checks
- **Composition over Inheritance**: Avoid deep class hierarchies
- **Interface-First**: Infrastructure adapters implement Core/Application interfaces
- **Descriptive Names**: `lastSeenAt` not `date`
- **camelCase Everywhere**: Including LIVR rules (`macAddress` not `mac_address`)
- **No Magic Numbers**: Extract numeric literals into named constants with JSDoc comments (`HISTORY_WINDOW_MS`, `TELEGRAM_MESSAGE_MAX_LENGTH`). Thresholds, limits, and configuration values must never appear inline.
- **Single Responsibility Methods**: Keep methods focused on one concern. Extract secondary logic (truncation, pagination, retry) into separate private methods that can be tested and reasoned about independently.

## Code Formatting Rules

### TypeScript Configuration

- **Module System**: `"module": "nodenext"`, `"moduleResolution": "nodenext"`
- **Target**: ES2022
- **Strict Mode**: All strict flags enabled
- **Compiler Options**:
  - `noUnusedLocals: true`
  - `noImplicitReturns: true`
  - `noFallthroughCasesInSwitch: true`
  - `noImplicitOverride: true`

### Import/Export Conventions

- **Always use `.js` extensions** in imports (TypeScript with NodeNext module resolution)
  - `import { User } from './user.entity.js';`
  - `import { BaseService } from '../../base-service.js';`
- **Use `type` imports** for types/interfaces: `import type { Device } from '@home-pulse-watcher/core';`
- **Named exports only** - avoid default exports
- **Barrel exports** through index.ts files for clean API surfaces

### Prettier Configuration

- **Single quotes**: `'string'` not `"string"`
- **Default Prettier rules** for everything else (2-space indent, trailing commas, etc.)

### Naming Conventions

- **Interfaces**: Prefix with `I` for repository/service abstractions (`IDeviceRepository`, `IEventEmitter`)
- **Types**: PascalCase without prefix (`PowerStatus`, `DeviceRole`, `LivrRules`)
- **Classes**: PascalCase (`BaseService`, `PrismaDeviceRepository`, `CreateUserService`)
- **Files**: kebab-case (`device.repository.ts`, `create-user.service.ts`, `power-status.enum.ts`)
- **Constants**: SCREAMING_SNAKE_CASE for error codes (`DEVICE_ALREADY_REGISTERED`)
- **Enums**: Use `as const` objects over TypeScript enums

### Service Structure

```typescript
export interface ServiceNameInput {
  field: string;
}

export interface ServiceNameOutput {
  result: ResultType;
}

export class ServiceNameService extends BaseService<ServiceNameInput, ServiceNameOutput> {
  constructor(private readonly repository: IRepository) {
    super();
  }

  protected validationRules(): LivrRules {
    return {
      field: ['required', 'string'],
    };
  }

  protected async execute(params: ServiceNameInput, context: ServiceContext): Promise<ServiceNameOutput> {
    // Implementation
  }
}
```

### Repository Pattern

- **Plain TypeScript classes** in Infrastructure layer (no `@Injectable()`)
- **Constructor injection** of PrismaClient
- **Map Prisma models** to Domain Entities using mapper functions
- **Never expose** Prisma types outside Infrastructure layer

### Error Handling

- **Specific error classes**: `ValidationError`, `DomainError`, `NotFoundError`
- **Error codes as constants**: `DomainErrorCode.DEVICE_ALREADY_REGISTERED`
- **Descriptive messages**: Include identifiers in error messages

### Documentation

- **JSDoc comments** for:
  - All public classes and interfaces
  - All public methods
  - Complex business logic
- **Concise descriptions**: One-line summary preferred
- **Include parameter/return descriptions** for non-obvious cases

### Keeping CHANGELOG.md in Sync

After completing a feature, bug fix, or significant change — add or update an entry in [`CHANGELOG.md`](CHANGELOG.md). Each entry should be a concise summary under a descriptive heading. Update existing entries if enhancing a previously logged feature.

### Keeping PROJECT_CONTEXT.md in Sync

After any change that affects the project's architecture, domain rules, tech stack, deployment, or historical context — update [`PROJECT_CONTEXT.md`](PROJECT_CONTEXT.md) to reflect the new state. This file is the shared mental model loaded at the start of every AI session.

Examples of changes that require a PROJECT_CONTEXT.md update:

- New or removed layers, modules, or significant services
- Changes to domain rules (debounce thresholds, HMAC parameters, ADC ranges, etc.)
- Deployment or infrastructure changes
- New historical incidents worth remembering (like the Flapping Incident)
- Stack or dependency upgrades that change developer workflow

<!-- nx configuration start-->
<!-- Leave the start & end comments to automatically receive updates. -->

## General Guidelines for working with Nx

- For navigating/exploring the workspace, invoke the `nx-workspace` skill first - it has patterns for querying projects, targets, and dependencies
- When running tasks (for example build, lint, test, e2e, etc.), always prefer running the task through `nx` (i.e. `nx run`, `nx run-many`, `nx affected`) instead of using the underlying tooling directly
- Prefix nx commands with the workspace's package manager (e.g., `pnpm nx build`, `npm exec nx test`) - avoids using globally installed CLI
- You have access to the Nx MCP server and its tools, use them to help the user
- For Nx plugin best practices, check `node_modules/@nx/<plugin>/PLUGIN.md`. Not all plugins have this file - proceed without it if unavailable.
- NEVER guess CLI flags - always check nx_docs or `--help` first when unsure

## Scaffolding & Generators

- For scaffolding tasks (creating apps, libs, project structure, setup), ALWAYS invoke the `nx-generate` skill FIRST before exploring or calling MCP tools

## When to use nx_docs

- USE for: advanced config options, unfamiliar flags, migration guides, plugin configuration, edge cases
- DON'T USE for: basic generator syntax (`nx g @nx/react:app`), standard commands, things you already know
- The `nx-generate` skill handles generator discovery internally - don't call nx_docs just to look up generator syntax

<!-- nx configuration end-->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vaReliy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
