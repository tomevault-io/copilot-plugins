## blinko

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Blinko is an open-source, self-hosted note-taking application with AI-powered features. It's a multi-platform application (web, desktop via Tauri, mobile) built with TypeScript/React frontend and Node.js/Express backend.

## Tech Stack

- **Frontend**: React 18, TypeScript, Vite, TailwindCSS, Tauri (for desktop apps)
- **Backend**: Node.js, Express, tRPC, Prisma ORM
- **Database**: PostgreSQL
- **Package Manager**: Bun (v1.2.8+)
- **Build Tool**: Turbo (monorepo management)
- **AI**: Multiple AI providers (OpenAI, Anthropic, Google, Azure, Ollama, etc.)

## Project Structure

```
blinko/
├── app/                    # Frontend React application
│   ├── src/               # React source code
│   ├── src-tauri/         # Tauri desktop app configuration
│   └── tauri-plugin-blinko/ # Custom Tauri plugin
├── server/                 # Backend Node.js server
│   ├── aiServer/          # AI integration services
│   ├── routerTrpc/        # tRPC API routes
│   └── routerExpress/     # Express API routes
├── prisma/                # Database schema and migrations
├── shared/                # Shared utilities and types
└── blinko-types/         # Type definitions
```

## Common Development Commands

### Setup & Installation
```bash
bun install                # Install dependencies
bun run prisma:generate    # Generate Prisma client
bun run prisma:migrate:dev # Run database migrations
```

### Development
```bash
bun run dev                # Run Tauri desktop app in development
bun run dev:backend        # Run backend server only
bun run dev:frontend       # Run frontend only
bun run prisma:studio      # Open Prisma Studio for database management
```

### Building
```bash
bun run build:web          # Build web application
bun run tauri:desktop:build # Build desktop application
bun run tauri:android:build # Build Android application
```

### Database
```bash
bun run prisma:migrate:deploy # Deploy migrations to production
bun run seed               # Seed database with initial data
```

### Testing & Linting
```bash
bun run test               # Run tests (if configured)
```

## Architecture & Key Components

### Frontend Architecture
- **State Management**: MobX with custom stores in `/app/src/store/`
- **Routing**: React Router v7
- **UI Components**: Custom components with HeroUI (@heroui/react)
- **Editor**: Vditor for markdown editing
- **Internationalization**: i18next with multiple language support
- **API Communication**: tRPC client for type-safe API calls

### Backend Architecture
- **API Layer**: Hybrid approach using both tRPC (type-safe) and Express routes
- **Authentication**: Multiple providers (local, OAuth via passport)
- **File Storage**: Local filesystem or S3-compatible storage
- **AI Integration**: Factory pattern for multiple AI providers
- **Background Jobs**: Cron-based scheduled tasks in `/server/jobs/`
- **Embeddings**: RAG (Retrieval-Augmented Generation) support with @mastra/rag

### Database Schema
- **Main Entities**: accounts, notes, attachments, tags, comments, conversations
- **ORM**: Prisma with PostgreSQL
- **Migrations**: Managed through Prisma migrate

## Environment Configuration

Create a `.env` file in the root directory with:
```
DATABASE_URL=postgresql://user:password@localhost:5432/blinko
NEXTAUTH_SECRET=your-secret-key
NEXTAUTH_URL=http://localhost:1111

# Optional S3 storage
S3_ENDPOINT=
S3_REGION=
S3_BUCKET=
S3_ACCESS_KEY=
S3_SECRET_KEY=

# AI Providers (optional)
OPENAI_API_KEY=
ANTHROPIC_API_KEY=
# ... other AI provider keys
```

## Important Patterns

1. **File Operations**: Use the filesystem routes in `/server/routerExpress/file/` for file handling
2. **AI Features**: AI providers are configured in `/server/aiServer/providers/`
3. **Type Safety**: Use tRPC routes when possible for type-safe API calls
4. **State Management**: Follow MobX patterns in store files
5. **Component Structure**: React components follow a modular structure with separate index.tsx files

## Deployment

### Docker
```bash
docker-compose -f docker-compose.prod.yml up -d
```

### Manual Deployment
1. Build the application: `bun run build:web`
2. Run migrations: `bun run prisma:migrate:deploy`
3. Start the server: `bun run start`

## Port Configuration
- Frontend/Full App: 1111 (default)
- Backend API: Same port (integrated with Vite Express)

## Mobile Development (Tauri)
- Android development: `bun run tauri:android:dev`
- iOS support through Tauri configuration
- Custom plugin in `/app/tauri-plugin-blinko/`

## Key Dependencies Notes
- Uses Bun as package manager and runtime
- Requires Node.js >= 20.0.0
- PostgreSQL database required
- Tauri requires Rust toolchain for desktop builds

---
> Source: [blinkospace/blinko](https://github.com/blinkospace/blinko) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
