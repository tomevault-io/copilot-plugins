## project-conventions

> Terragon AI coding assistant platform development conventions and guidelines

# Terragon Project Conventions

This document outlines key conventions and commands for working with the Terragon AI coding assistant platform monorepo.

## Project Overview

Terragon is an AI-powered coding assistant platform that integrates Claude AI with real development environments through multi-provider sandbox orchestration, GitHub workflows, and real-time collaboration.

## Package Management

- **Always use `pnpm v10.11.1`** for package management operations.
  ```bash
  pnpm add <package-name>
  pnpm install
  ```

## Monorepo Structure

This is a sophisticated monorepo with multiple apps and packages:

### Core Applications
- `apps/www` - Next.js 15.3.3 frontend with React 19
- `apps/broadcast` - PartyKit WebSocket service for real-time features

### Shared Packages
- `@terragon/shared` - Database models, schemas, core utilities
- `@terragon/daemon` - Sandbox agent runtime and communication
- `@terragon/bundled` - Bundled scripts for deployment (includes the daemon cli and mcp server)
- `@terragon/env` - Environment configuration management
- `@terragon/r2` - Cloudflare R2 storage integration
- `@terragon/dev-env` - Docker development environment
- `@terragon/tsconfig` - Shared TypeScript configuration

## Development Commands

### Start Development Environment
```bash
# Start full development environment (env, www, cron, broadcast, daemon services)
pnpm dev

# Frontend development only
cd apps/www && pnpm dev

# WebSocket/broadcast development
cd apps/broadcast && pnpm dev
```

### Testing & Quality
```bash
# Run tests (www app with Vitest)
cd apps/www && pnpm test

# Run linting
cd apps/www && pnpm lint

# TypeScript type checking
cd apps/www && pnpm check

# TypeScript watch mode for all services
pnpm tsc-watch
```

### Database Operations (Drizzle ORM)
```bash
# Push schema to dev database
cd apps/www && pnpm drizzle-kit-push-dev

# Open database studio
cd apps/www && pnpm drizzle-kit-studio-dev
```

### Deployment
```bash
# Build frontend (includes daemon packages)
cd apps/www && pnpm build

# Deploy broadcast service to PartyKit
cd apps/broadcast && pnpm deploy
```

## Frontend (apps/www)

### Technology Stack
- **Framework**: Next.js 15.3.3 (App Router) with React 19
- **Styling**: Tailwind CSS v4 with PostCSS
- **State Management**: Jotai
- **UI Components**: Radix UI primitives with shadcn/ui
- **API**: Hono framework with tRPC
- **Real-time**: PartySocket for WebSocket communication

### Adding Shadcn UI Components
1. Navigate to the www directory:
   ```bash
   cd apps/www
   ```
2. Add components:
   ```bash
   npx shadcn@latest add <component-name> --yes
   ```

### Code Conventions
- Use TypeScript throughout with strict typing
- Prefer server components over client components
- Use Jotai for client-side state management
- Follow established patterns in `src/components/`
- Implement proper error handling and loading states
- Use workspace packages (`@terragon/*`) for shared code

## Architecture Components

### Core Agent System (`apps/www/src/agent/`)
- **Sandbox Orchestration**: Multi-provider management (E2B, CodeSandbox, Local)
- **Claude Integration**: Message processing and tool execution
- **Daemon System**: Autonomous Node.js agent running in sandboxes
- **Resource Management**: Hibernation and session lifecycle management

### Chat System (`apps/www/src/components/chat/`)
- **Real-time UI**: React-based chat interface with message streaming
- **Tool Visualization**: Custom components for bash, file operations, git diff
- **Message Processing**: Structured Claude message handling with tool calls
- **Status Tracking**: Thread status, sandbox state, and real-time updates

### Database Schema (`packages/shared/src/db/`)
- **Threads**: Chat sessions with sandbox and GitHub integration
- **Users**: Authentication, settings, and API key management
- **Environments**: User-repository combinations for development
- **GitHub PRs**: Pull request tracking and automated workflows

## Key Dependencies

### AI & Sandbox Integration
- `@anthropic-ai/sdk` v0.52.0 - Claude AI integration
- `@codesandbox/sdk` v1.1.6 - CodeSandbox provider
- `@e2b/code-interpreter` (beta) - E2B provider
- `ai` v4.3.16 - AI SDK for streaming

### Database & Auth
- `drizzle-orm` v0.43.1 - Database ORM
- `better-auth` v1.2.8 - Authentication with GitHub OAuth
- `@upstash/ratelimit` v2.0.5 - Rate limiting with Redis

### GitHub Integration
- `octokit` v5.0.2 - GitHub API client
- `@ai-sdk/openai` v1.3.22 - OpenAI for commit messages

## File Naming Conventions

- Use kebab-case for file names
- Use PascalCase for React components
- Use camelCase for TypeScript interfaces/types
- Add `.test.ts` suffix for test files
- Use `.mdc` for Cursor documentation files

## Important Patterns

### Sandbox System
- Multiple providers with unified interface
- Resource hibernation for cost optimization
- Autonomous daemon communication
- Real-time status updates

### GitHub Workflows
- Automated branch creation and commits
- AI-generated commit messages and PR descriptions
- Real-time PR status synchronization

### Security Guidelines
- Never commit API keys or secrets
- Use proper rate limiting with Upstash
- Validate all inputs with Zod schemas
- Implement proper authentication checks
- Secure sandbox isolation

## Environment Setup

### Required Environment Variables
- **AI Services**: Anthropic Claude, OpenAI, CodeSandbox, E2B API keys
- **Infrastructure**: Database URL, Redis, Cloudflare R2 storage
- **Auth**: GitHub OAuth credentials, internal shared secrets
- **Development**: ngrok for local tunnel, sandbox provider configs

### Configuration Files
- `.env.example` files in apps/www, apps/broadcast, packages
- Docker Compose with PostgreSQL 16, Redis 7
- TypeScript shared config via `@terragon/tsconfig`

## Reference Files

- `CLAUDE.md` - Comprehensive project documentation
- `apps/www/src/agent/sandbox.ts` - Core sandbox orchestration
- `apps/www/src/components/chat/chat-ui.tsx` - Main chat interface
- `packages/shared/src/db/schema.ts` - Database schema definitions
- `apps/www/src/lib/auth-server.ts` - Authentication utilities

## Development Workflow

1. **Thread Lifecycle**: User creates thread → Sandbox provisioning → Daemon installation → Claude processing → Git integration → Resource cleanup
2. **Real-time Collaboration**: WebSocket-based live updates via PartyKit
3. **Resource Management**: Cost optimization through hibernation
4. **Mobile Support**: Full-featured responsive design

---
> Source: [terragon-labs/terragon-oss](https://github.com/terragon-labs/terragon-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
