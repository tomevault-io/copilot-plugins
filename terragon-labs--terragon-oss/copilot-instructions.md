## terragon-oss

> You are an AI assistant helping with the Terragon codebase, an AI-powered coding assistant platform that allows users to run coding agents in parallel inside remote sandboxes.

# Terragon Project Cursor Rules

You are an AI assistant helping with the Terragon codebase, an AI-powered coding assistant platform that allows users to run coding agents in parallel inside remote sandboxes.

## Project Overview

Terragon is a sophisticated monorepo that enables concurrent AI coding tasks in full development environments with capabilities for editing, testing, and creating PRs.

## Repository Structure & Conventions

### Monorepo Setup
- Uses **pnpm workspace (v10.11.1)** - ALWAYS use `pnpm` for package management
- TypeScript throughout with strict typing
- Path aliases: `@/*` → `./src/*` in Next.js app
- Workspace dependencies use `workspace:*` references

### Core Applications
- `apps/www` - Next.js 15.3.4 frontend with React 19.1
- `apps/broadcast` - PartyKit WebSocket service for real-time features
- `apps/docs` - Fumadocs-based documentation site  
- `apps/cli` - Terry CLI tool (uses Ink for terminal UI)

### Shared Packages
- `@terragon/shared` - Database models, schemas, core utilities
- `@terragon/daemon` - Sandbox agent runtime and communication
- `@terragon/bundled` - Bundled deployment scripts
- `@terragon/env` - Environment configuration management
- `@terragon/r2` - Cloudflare R2 storage integration
- And 8+ other specialized packages

## Technology Stack

### Frontend (apps/www)
- **Framework**: Next.js 15.3.4 (App Router) with React 19
- **Styling**: Tailwind CSS v4
- **State**: Jotai, React Query (Tanstack Query)
- **UI Components**: Radix UI primitives with shadcn/ui
- **Rich Text**: TipTap editor with slash commands
- **Real-time**: PartySocket for WebSocket communication

### Backend & Database
- **Database**: Drizzle ORM with PostgreSQL
- **Auth**: Better Auth with GitHub OAuth, Claude OAuth integration
- **AI**: Anthropic Claude SDK (v0.52.0), AI SDK, OpenAI
- **Rate Limiting**: @upstash/ratelimit with Redis
- **File Storage**: Cloudflare R2 integration

### Development & Testing
- **Testing**: Vitest
- **CLI**: Ink for interactive terminal UI, ORPC for type-safe API communication
- **Email**: React Email for transactional emails
- **UI Development**: Ladle for component development

## Key Development Commands

```bash
# Start full development environment
pnpm dev

# TypeScript watch mode for all services  
pnpm tsc-watch

# Run tests
pnpm -C apps/www test
pnpm -C packages/shared test

# Database operations
pnpm -C packages/shared drizzle-kit-push-dev
pnpm -C packages/shared drizzle-kit-studio-dev

# CLI tool development
pnpm install-cli:dev
```

## Code Conventions

### React/Next.js
- Prefer server components over client components
- Use Jotai for client-side state management
- Follow established patterns in `src/components/`
- Implement proper error handling and loading states
- Use workspace packages (`@terragon/*`) for shared code

### Database
- Use Drizzle ORM patterns
- Push schema changes with `drizzle-kit-push-dev`
- All environment configs in dedicated files

### File Naming
- Use kebab-case for file names
- Use PascalCase for React components
- Use camelCase for TypeScript interfaces/types
- Add `.test.ts` suffix for test files
- Use `.mdc` for Cursor documentation files

## Feature Flags System

### Creating Feature Flags
1. Define in `packages/shared/src/model/feature-flags-definitions.ts`
2. Use `useFeatureFlag` hook in React components
3. Configure via admin page (global + per-user overrides)

### Example Usage
```typescript
import { useFeatureFlag } from "@/hooks/use-feature-flag";

const MyComponent = () => {
  const isFeatureEnabled = useFeatureFlag("myNewFeature");
  // Use the flag...
};
```

## Architecture Components

### Core Agent System (`apps/www/src/agent/`)
- Sandbox orchestration (E2B, Docker)
- Claude integration and message processing
- Daemon system for autonomous agents
- Resource management and hibernation

### Chat System (`apps/www/src/components/chat/`)
- Real-time UI with message streaming
- Tool visualization (bash, files, git diff, web search, todos)
- Rich text editor with mentions and attachments
- Follow-up tasks and copy features

### Database Schema (`packages/shared/src/db/`)
- Threads with sandbox/GitHub integration
- Users with auth and API key management
- Environments with variables and MCP config
- GitHub PRs with automated workflows
- Feature flags and user preferences

## Environment Configuration

### Development Setup
- PostgreSQL on port 5432, Redis on port 6379
- Test DB on port 15432, Test Redis on port 16379
- Use `.env.development.local` for local development
- Docker Compose handles dev infrastructure

### Key Environment Variables
- AI Services: Anthropic Claude, OpenAI, E2B API keys
- Infrastructure: Database URL, Redis, Cloudflare R2
- Auth: GitHub OAuth, internal shared secrets
- Development: ngrok tunnel, sandbox provider configs

## Security Guidelines
- Never commit API keys or secrets
- Use proper rate limiting with Upstash
- Validate all inputs with Zod schemas
- Implement proper authentication checks
- Secure sandbox isolation

## Recent Features to be aware of
- Claude OAuth integration
- Follow-up task queuing
- Interactive CLI with thread selection
- MCP Server for task suggestions  
- Task sharing with visibility controls
- Access codes for controlled signup
- Slash commands in TipTap editor
- Admin UI enhancements

## Troubleshooting Common Issues
- Docker not starting: Check ports 5432, 6379 are free
- TypeScript errors: Run `pnpm install` after pulling
- DB schema out of sync: Run `drizzle-kit-push-dev`
- Sandbox issues: Verify E2B API keys
- Ngrok issues: Check auth token and domain in env

## When Working on This Codebase
1. Always use `pnpm` for package management
2. Check existing patterns in similar components before implementing
3. Use workspace packages for shared functionality
4. Implement proper TypeScript typing
5. Follow the established file naming conventions
6. Test database changes with push commands
7. Consider feature flags for new functionality
8. Maintain security best practices

---
> Source: [terragon-labs/terragon-oss](https://github.com/terragon-labs/terragon-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
