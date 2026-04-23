## zen-send

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Zen Send - Cross-platform clipboard, text, and file transfer tool (like LocalSend)

## Project Overview

Zen Send is a monorepo containing a server, web frontend, and React Native mobile app for cross-device content transfer.

## Architecture

### Apps
- **apps/server**: Express.js backend with Socket.io for real-time communication
- **apps/web**: React 19 frontend with Vite (port 5274, proxies API to server)
- **apps/mobile**: React Native (Expo) for Android and iOS
- **apps/electron**: Electron desktop app for Windows/macOS/Linux

### Packages
- **packages/dto**: Shared TypeScript interface types for request/response DTOs (RegisterRequest, LoginRequest, etc.)
- **packages/shared**: Re-exports from `@zen-send/dto` plus domain types (Device, TransferSession, Socket events)
- **packages/logger**: Pino-based logging utility

### Config
- **config/eslint-config**: Shared ESLint configuration
- **config/typescript-config**: Shared TypeScript configurations

## Commands

```bash
pnpm install          # Install all dependencies
pnpm dev              # Run all apps in dev mode
pnpm dev:web          # Run web frontend (port 5274)
pnpm dev:server       # Run backend server (port 3110)
pnpm dev:mobile       # Run Expo mobile app
pnpm build            # Build all packages
pnpm clean            # Clean all dist outputs
pnpm lint             # Run ESLint
pnpm lint:fix         # Auto-fix ESLint issues
pnpm format           # Format code with Prettier
pnpm typecheck        # Run TypeScript type checking

# Server-specific
pnpm --filter @zen-send/server dev    # Run server only
pnpm --filter @zen-send/server build  # Build server
pnpm --filter @zen-send/server typecheck
pnpm --filter @zen-send/server migrate:generate  # Generate Drizzle migrations
pnpm --filter @zen-send/server migrate:migrate    # Run Drizzle migrations

# Web-specific
cd apps/web && pnpm dev          # Run web only
cd apps/web && pnpm build        # Build web for production (outputs to server/public)
cd apps/web && pnpm typecheck   # Type-check web only

# Electron-specific
cd apps/electron && pnpm dev     # Run Electron app in dev mode
cd apps/electron && pnpm dist:mac  # Build macOS .app
cd apps/electron && pnpm dist:win  # Build Windows .exe
cd apps/electron && pnpm dist:linux  # Build Linux AppImage
```

## Code Organization

### Server (Express + routing-controllers + Socket.io)

```
apps/server/src/
├── index.ts              # Entry point
├── app.ts                # App factory (creates Express + Socket.io)
├── ioc.ts                # Manual IOC container setup using glob patterns
├── controllers/           # routing-controllers @JsonController classes
├── services/             # typedi @Service classes (business logic)
├── validators/           # class-validator DTOs (runtime validation)
├── middlewares/          # Auth middleware
├── socket/               # Socket.io event handlers
├── db/                   # Drizzle ORM schema
└── utils/                # JWT, ID generation, response helpers
```

**Server Architecture Key Points:**
- Uses `routing-controllers` for declarative API endpoints with validation
- Uses `typedi` for dependency injection (IOC via glob-loaded services/controllers)
- Request validation: `validators/*.validator.ts` classes with class-validator decorators
- Type definitions: `packages/dto` for compile-time interfaces (server imports from `@zen-send/dto`)
- `AuthService`, `DeviceService`, `TransferService` contain business logic
- `DbService` wraps database operations
- `S3Service` handles S3 presigned URLs

**Server IOC Rules:**
- `container.ts` must be imported before all controllers (ensures TypeDI init first)
- Services use `@Service()` decorator and constructor injection
- Socket handlers use `Container.get()` (not constructor injection)

### Transfer Module (Chunked S3 Multipart Upload)
- Files are uploaded in 1MB chunks via S3 multipart upload
- Server generates presigned URLs for direct client-to-S3 upload
- Tracks chunk uploads in `chunkUploads` table
- Supports text and clipboard transfers in addition to files
- Transfer sessions expire after `TRANSFER_TTL_DAYS` (default 30)

### Database Schema (Drizzle ORM + MySQL)
**Tables:** `users`, `devices`, `transfer_sessions`, `transfer_items`, `download_history`, `chunk_uploads`
- **No foreign keys** - Joins done in business code
- **Unix timestamps** - All timestamps stored as integers (seconds, not milliseconds)
- **Schema location:** `apps/server/src/db/schema.ts`

### DTO Architecture

**Two-layer DTO system:**
1. `packages/dto` - Pure TypeScript interfaces for compile-time type checking
2. `apps/server/src/validators/` - class-validator decorated classes for runtime validation

```
packages/dto/src/index.ts      → interface LoginRequest { email: string; password: string }
apps/server/validators/        → class LoginDto implements LoginRequest { @IsEmail() email!: string }
```

Web imports types from `@zen-send/dto`, server imports types and adds validation decorators.

### Web App (React 19 + Vite + Tailwind CSS 4 + @rabjs/react)
- **Styling**: Tailwind CSS v4 with CSS variables for theming
- **State Management**: @rabjs/react for reactive state with observer/Service patterns
- **Theme System**: `src/theme/` - Light/dark mode via CSS variables
- **Real-time**: Socket.io client for device discovery and transfers
- **Web build output**: `apps/server/public/` (served by Express in production)

### @rabjs/react Critical Rules
These rules are **non-obvious and must be followed**:

1. **Components must use `observer()`** - Components wrapped with `observer()` from @rabjs/react才能响应状态变化
2. **Never destructure observables** - `const { count } = service` breaks reactivity; use `service.count` directly
3. **resolve() must use getters** - Use `get apiService() { return this.resolve(ApiService); }` not property assignment
4. **Global vs page services** - Global: `register()` in main.tsx; Page-level: `bindServices()` at component export
5. **API types** - ApiService unwraps `data` layer; type generics reflect actual structure, not wrapper

### Real-time Communication (Socket.io)
**Client → Server events:**
- `device:heartbeat` - Keep device marked as online
- `device:register` - Explicit device registration
- `transfer:notify` - Send transfer notification to target device
- `transfer:progress` - Emit progress updates to session room
- `transfer:complete` - Notify session of transfer completion

**Server → Client events:**
- `device:list` - List of user's devices (online/offline status)
- `transfer:new` - New incoming transfer notification

### ID Generation
Uses `nanoid` with 22-character IDs and type prefixes:
| Prefix | Entity | Example |
|--------|--------|---------|
| `u` | User | `u3KkL9mW2XyPqRsTuVwY` |
| `d` | Device | `d4LlMnO5PqRsTuVwXyZa` |
| `s` | Transfer Session | `s5MmNoO6QrStUvWxYbZc` |
| `i` | Transfer Item | `i6NnOpP7RsTuVwXyZcAd` |
| `h` | Download History | `h7OoPqQ8StUvWxYzAdBe` |
| `c` | Chunk | `c8PpQrR9TuVwXyZaBdCe` |

## Naming Conventions

- **Files and folders**: Use lowercase English with hyphens as separators (e.g., `feature-name`, `use-auth.ts`)
- **No camelCase** in file or directory names
- **Validators**: `*.validator.ts` suffix (e.g., `auth.validator.ts`)
- **Controllers**: `*.controller.ts` suffix

### Component Organization (for pages/*/components/)

```
components/
├── feature-name/                 # Feature module (all related code together)
│   ├── index.ts                 # Barrel export for all public APIs
│   ├── feature-name.ts          # Main component
│   ├── feature-name.service.ts  # Service class (if using service pattern)
│   ├── feature-name-header.tsx  # Sub-components (header, footer, etc.)
│   ├── feature-name-content.tsx
│   ├── helper-component.tsx      # Helper components used only by this feature
│   └── hooks/                   # Custom hooks
│       └── use-hook.ts
```

## Tech Stack

- **Package Manager**: pnpm with workspaces
- **Build Tool**: Turbo
- **Server**: Express.js + routing-controllers + Socket.io + typedi
- **Web**: React 19 + Vite + Tailwind CSS v4 + @rabjs/react
- **Mobile**: React Native + Expo
- **Desktop**: Electron 40 + Vite + @rabjs/react
- **Validation**: class-validator + class-transformer
- **Database**: Drizzle ORM + MySQL
- **File Storage**: AWS S3 (presigned URLs for direct client upload)

## Development

1. Copy `.env.example` to `.env` in `apps/server`
2. Run `pnpm install`
3. Run `pnpm dev:server` to start the backend (auto-reload on file changes)
4. Run `pnpm dev:web` for frontend (runs on port 5274, proxies API to server port)

### Environment Variables

**Server (`apps/server/.env`):**
```
PORT=3110
NODE_ENV=development

# JWT (required in production)
JWT_ACCESS_SECRET=<your-access-secret>
JWT_REFRESH_SECRET=<your-refresh-secret>

# S3 (for file transfers)
S3_REGION=us-east-1
S3_ENDPOINT=<s3-compatible-endpoint>  # Optional, for S3-compatible storage
S3_ACCESS_KEY_ID=<key>
S3_SECRET_ACCESS_KEY=<secret>
S3_BUCKET=zen-send-transfers
TRANSFER_TTL_DAYS=30
```

### Web-Specific
- **Tailwind CSS v4**: Uses `@tailwindcss/postcss` plugin with PostCSS
- **Theme Tokens**: Design tokens in `src/theme/tokens.ts` are applied as CSS variables on `:root`
- **Dark Mode**: Controlled via `dark` class on `<html>` element
- **@rabjs/react**: Service-based state management using observer/view patterns with dependency injection

### Git Hooks
- **commitlint** validates commit messages (Conventional Commits format) on commit
- **changesets** manages versioning and changelogs (run `pnpm changeset` to create)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ximing) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
