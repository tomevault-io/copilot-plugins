## infinitenovel

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Core Development
```bash
# Development server
npm run dev

# Production build (includes Prisma generation)
npm run build

# Start production server
npm start

# Linting
npm run lint

# Testing
npm run test
npm run test:ui  # Vitest UI mode
```

### Database Management
```bash
# Generate Prisma client
npx prisma generate

# Create and apply migrations
npx prisma migrate dev

# View database in browser
npx prisma studio
```

## Architecture Overview

### Novel Generation Pipeline
The platform implements a sophisticated multi-stage AI generation process:

1. **Main Outline Generation**: Creates overarching narrative structure from user input
2. **World Building**: Generates characters, scenes, and plot elements stored in vector database
3. **Detailed Chapter Outlines**: Breaks down story into chapter-level details
4. **Content Generation**: Produces actual chapter content with context retrieval

### Vector Memory System (RAG)
- **ChromaDB Integration**: Stores world-building elements as vector embeddings
- **Semantic Search**: Retrieves relevant context for each generation to maintain narrative consistency
- **Located in**: `/src/lib/vector-store.ts`

### Multi-Provider AI Support
- **Abstraction Layer**: Unified interface in `/src/lib/ai-client.ts` supporting OpenAI and Google Gemini
- **Retry Logic**: Robust error handling with exponential backoff
- **Streaming Support**: Real-time content generation

## Key Directories & Files

### Core Application Structure
- `/src/app/` - Next.js App Router pages and API routes
  - `/api/` - API endpoints for novels, chapters, and AI operations
  - `/create/` - Novel creation interface
  - `/novels/` - Novel management and reading
  - `/settings/` - AI model configuration

- `/src/components/` - Reusable React components
  - `/ui/` - shadcn/ui base components
  - `/create/` - Novel creation components
  - `/novel/` - Novel-specific components
  - `/layout/` - Layout components (Sidebar, etc.)

- `/src/lib/` - Core business logic
  - `/ai/` - AI client implementations
  - `/generation/` - Novel generation logic
  - `/prompts/` - Structured AI prompts for different tasks
  - `/utils/` - Utility functions and helpers

### Database Schema
- **Novel**: Main entity with metadata, outlines, and word count tracking
- **NovelChapter**: Individual chapters with content and numbering
- **NovelRole**: Character management with unique names per novel
- **NovelScene**: Location and setting management
- **NovelClue**: Plot elements and foreshadowing

## Configuration

### Environment Variables
Required in `.env` file:
- `OPENAI_BASE_URL` - AI provider base URL
- `OPENAI_API_KEY` - API key for AI services
- `DATABASE_URL` - PostgreSQL connection string

### AI Model Configuration
- Flexible provider setup through settings UI
- Supports OpenAI and Google Gemini
- Configurable through `/src/app/settings/` interface

## Code Patterns

### State Management
- **Zustand**: Client-side state management
- **React Hook Form + Zod**: Form validation and handling
- **Server Actions**: Database operations through Next.js API routes

### Type Safety
- Comprehensive TypeScript definitions in `/src/types/`
- Zod schema validation for API endpoints
- Prisma-generated types for database entities

### Error Handling
- Centralized logging with Pino
- Retry mechanisms for AI API calls
- Graceful degradation for vector database operations

## Testing & Quality

### Testing Setup
- **Vitest**: Primary testing framework with UI support
- **Test location**: Files should be placed alongside source files with `.test.ts` extension
- **Run tests**: `npm run test` or `npm run test:ui` for interactive testing

### Linting & Formatting
- **ESLint**: Next.js configuration with TypeScript support
- **Prettier**: Code formatting (already configured in package.json)
- **Run linting**: `npm run lint`

## Database Migrations

### Creating Migrations
```bash
npx prisma migrate dev --name descriptive_migration_name
```

### Resolving Migration Issues
```bash
npx prisma migrate resolve --applied migration_name
npx prisma migrate resolve --rolled-back migration_name
```

## Important Notes

### Build Process
- The build script automatically runs `prisma generate` before `next build`
- Always ensure database schema is up-to-date before building
- Vector database (ChromaDB) operations have fallbacks for production resilience

### AI Integration
- All AI operations go through the abstraction layer in `/src/lib/ai-client.ts`
- Prompts are structured and located in `/src/lib/prompts/`
- Vector store operations handle connection errors gracefully

### Performance Considerations
- Large novels may require pagination for chapter lists
- Vector search operations are optimized for narrative context retrieval
- AI streaming provides real-time feedback during content generation

---
> Source: [SliverKeigo/infinitenovel](https://github.com/SliverKeigo/infinitenovel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
