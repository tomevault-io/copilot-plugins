## react-native-vibe-code-sdk

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

React Native Vibe Code is an open-source AI-powered IDE for building React Native and Expo mobile applications through natural language. Users describe what they want to build and the AI generates the code in secure sandboxes.

## Stack

- **Frontend**: Next.js 14 with App Router, TypeScript, Tailwind CSS, shadcn/ui components
- **Backend**: Next.js API routes, PostgreSQL with Drizzle ORM
- **AI**: Anthropic Claude integration via Claude Code SDK
- **Sandboxing**: E2B Code Interpreter for secure code execution
- **Authentication**: Better Auth with Google OAuth
- **Subscriptions**: Polar integration for billing and subscriptions
- **Real-time**: Pusher for live updates and file watching
- **Deployment**: Designed for Vercel with Neon PostgreSQL

## Development Commands

### Core Development
```bash
# Development server on port 3210
pnpm run dev:3210

# Build production version
pnpm run build

# Start production server
pnpm start

# Type checking
pnpm run type-check

# Linting
pnpm run lint
```

### Database Management
```bash
# Generate migration files
pnpm run db:generate

# Run migrations
pnpm run db:migrate

# Push schema changes directly (development)
pnpm run db:push

# Open Drizzle Studio
pnpm run db:studio
```

### Sandbox Template Building
```bash
# Build Expo sandbox template
pnpm run build:sandbox:expo

# Build Tamagui sandbox template
pnpm run build:sandbox:tamagui

# Build GitHub integration template
pnpm run build:sandbox:github
```

### Utilities
```bash
# Run Inngest development server
pnpm run inngest:dev

# Clean up Docker resources
pnpm run docker:prune

# Kill all E2B sandboxes
pnpm run e2b:kill-all
```

## Architecture

### Core Application Structure

- **app/**: Next.js App Router pages and API routes
  - **api/**: Backend API endpoints for chat, sandbox management, authentication, file operations
  - **p/[id]/**: Project pages for individual sandbox instances
- **components/**: Reusable UI components built with shadcn/ui
- **lib/**: Core business logic, utilities, and services
  - **db/**: Database schema and migrations using Drizzle ORM
  - **auth/**: Better Auth configuration and helpers
- **sandbox-templates/**: E2B Dockerfile templates for different environments

### Key Services

#### Sandbox Management (`lib/sandbox-file-watcher.ts`, API routes)
- Creates and manages isolated code execution environments
- Supports Expo React Native development with hot reload
- File watching system for real-time code updates
- Template-based sandbox creation (Expo, Tamagui, GitHub integration)

#### Chat System (`app/api/chat/route.ts`, `components/chat-*.tsx`)
- AI-powered conversation interface
- Integrates with Anthropic Claude via Claude Code SDK
- Message history and project context management
- Real-time streaming responses

#### File Operations (`app/api/file-*`, `lib/file-*.ts`)
- Real-time file watching and synchronization
- Code editing with Monaco Editor
- Git operations and version control
- Project structure management

#### Authentication & Authorization (`lib/auth/`)
- Better Auth with Google OAuth integration
- Session management and user context
- Rate limiting and usage tracking

#### Subscription Management (`lib/polar-client.ts`, Polar integration)
- Tiered subscription system (Free, Start, Pro, Senior)
- Usage-based billing and limits
- Webhook handling for subscription events

### Environment Configuration

Required environment variables:
- `ANTHROPIC_API_KEY`: Claude AI integration
- `DATABASE_URL`: PostgreSQL connection string
- `BETTER_AUTH_SECRET`: Authentication secret (min 32 chars)
- `E2B_API_KEY`: Sandbox environment access
- `PUSHER_*`: Real-time communication
- `POLAR_*`: Subscription and billing integration

Optional for full functionality:
- `GOOGLE_CLIENT_ID/SECRET`: OAuth authentication
- `DEEPGRAM_API_KEY`: Voice transcription
- `GITHUB_*`: GitHub integration features

### Database Schema

- **user**: Better Auth compatible user table with custom fields
- **session/account**: Authentication and OAuth data
- **project**: User projects and sandbox instances
- **message**: Chat conversation history
- **usage_tracking**: API usage and rate limiting
- **subscription**: User subscription status and limits

### Template System

Located in `sandbox-templates/`, each template includes:
- **e2b.Dockerfile**: Container setup and dependencies
- **e2b.toml**: Template configuration (memory, CPU, start commands)
- Helper scripts for specific frameworks (Expo, Tamagui)

## Key Development Patterns

### Error Handling
- Comprehensive error boundary system
- User-facing error notifications with technical details
- Debug endpoints for troubleshooting subscriptions and services

### Real-time Updates
- Pusher integration for live file changes
- Sandbox status monitoring
- Collaborative editing capabilities

### Code Generation
- AI prompt engineering in `prompt-engine/`
- Template-based code scaffolding
- Framework-specific optimizations

### Security
- Sandboxed code execution via E2B
- Rate limiting on API endpoints
- User input validation and sanitization
- Secure authentication flow

## Testing & Deployment

### Local Development
1. Install dependencies: `pnpm install`
2. Set up environment variables in `.env.local` (root directory)
3. Run setup to symlink env to web app: `pnpm setup`
4. Run database migrations: `pnpm run db:migrate`
5. Start development server: `pnpm run dev`

### Production Deployment (Vercel)
- Automatic deployments from main branch
- Environment variables configured in Vercel dashboard
- Database migrations run automatically via Inngest workers
- E2B sandbox templates must be built and deployed separately
- **IMPORTANT**: Do NOT add `outputDirectory` to `vercel.json`. Vercel auto-detects the Next.js app in `apps/web/` and resolves `outputDirectory` relative to it. Setting `outputDirectory: "apps/web/.next"` causes a double path error (`apps/web/apps/web/.next`). Let Vercel handle it automatically.

## Common Workflows

### Adding New AI Model Support
1. Update `lib/models.json` with model configuration
2. Add provider-specific handling in chat API route
3. Update UI model selector in `components/chat-picker.tsx`

### Creating New Sandbox Templates
1. Create template directory in `sandbox-templates/`
2. Add Dockerfile and e2b.toml configuration
3. Add build script to `package.json`
4. Update template registry in `lib/templates.json`

### Database Schema Changes
1. Modify schema in `lib/db/schema.ts`
2. Generate migration: `pnpm run db:generate`
3. Review and apply: `pnpm run db:migrate`
4. Update TypeScript types as needed

---
> Source: [react-native-vibe-code/react-native-vibe-code-sdk](https://github.com/react-native-vibe-code/react-native-vibe-code-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
