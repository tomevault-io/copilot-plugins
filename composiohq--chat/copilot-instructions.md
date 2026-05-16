## chat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `pnpm dev` - Start development server with Turbo
- `pnpm build` - Run database migrations and build production app
- `pnpm lint` - Run Next.js and Biome linting with auto-fix
- `pnpm format` - Format code with Biome
- `pnpm test` - Run Playwright e2e tests

## Database Commands

- `pnpm db:migrate` - Run database migrations
- `pnpm db:generate` - Generate Drizzle migrations
- `pnpm db:studio` - Open Drizzle Studio for database inspection
- `pnpm db:push` - Push schema changes to database

## Architecture

This is a Next.js AI chatbot application using:

### Core Stack

- **Next.js 15** with App Router and React Server Components
- **AI SDK** for LLM integration with Anthropic Claude models
- **Drizzle ORM** with PostgreSQL for data persistence
- **Auth.js** for authentication
- **shadcn/ui** components with Tailwind CSS

### Project Structure

- `/app/(auth)/` - Authentication routes and components
- `/app/(chat)/` - Main chat interface and API routes
- `/artifacts/` - Different artifact types (code, image, text, sheet)
- `/components/` - Reusable UI components
- `/lib/ai/` - AI model configuration and tools
- `/lib/db/` - Database schema and utilities
- `/hooks/` - Custom React hooks
- `/tests/` - Playwright e2e tests

### AI Integration

- Default model: `claude-4-sonnet-20250514` for chat, `claude-3-5-haiku-latest` for titles
- Configurable via `/lib/ai/providers.ts`
- Test environment uses mock models from `/lib/ai/models.test.ts`
- AI tools defined in `/lib/ai/tools/`

### Database Schema

- Uses Drizzle ORM with PostgreSQL
- Core tables: User, Chat, Message_v2 (new schema), Document, Vote, Suggestion
- Migration files in `/lib/db/migrations/`
- The Message table is deprecated in favor of Message_v2

### Key Features

- Real-time chat with artifacts (code, documents, images, spreadsheets)
- File upload and document processing
- Chat history with public/private visibility
- User voting and suggestions system
- Multi-modal input support

## Code Quality

- Uses Biome for linting and formatting
- ESLint for Next.js specific rules
- TypeScript with strict configuration
- Package manager: pnpm

## Development Best Practices

### Authentication Changes

- **Plan schema completely upfront** - Design all auth-related fields in single migration
- **Exclude auth routes from middleware** - Always exempt `/login`, `/register` from auth checks first
- **Use unique identifiers over emails** - OAuth provider IDs are more reliable than email addresses
- **Test complete auth flows** - Test end-to-end after each major auth change
- **Avoid type safety compromises** - Use proper optional types instead of non-null assertions

### Database Migrations

- **Single comprehensive migration over incremental** - Plan all related fields together
- **Include constraints from start** - Define unique, foreign key constraints in initial schema
- **Clean slate approach for development** - Clear test data when making major auth changes

### Middleware and Routing

- **Auth exclusions first** - Always handle public routes before implementing auth redirects
- **Avoid redirect loops** - Check middleware patterns don't redirect auth pages to themselves

### Error Handling

- **Detailed logging for auth flows** - Log profile data and validation steps for debugging
- **Graceful fallbacks** - Design for missing optional data (emails, names) from OAuth providers

### Code Quality During Development

- **Run `pnpm format` and `pnpm lint`** after implementing any granular part of functionality
- **Avoid type safety compromises** - No ts-ignores or lint ignores, use proper typing instead
- **Test incrementally** - Verify each step works before moving to the next

### Git Workflow

- Use gh cli when working with git stuff. If and when asked, make PRs using it too.

## Composio Documentation

- For composio related documentation, read composio-docs.md to find the relevant URLs to read the documentation from.

---
> Source: [ComposioHQ/chat](https://github.com/ComposioHQ/chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
