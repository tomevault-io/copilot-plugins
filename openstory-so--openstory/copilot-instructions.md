## context7-integration

> Context7 MCP integration for up-to-date documentation


# Context7 MCP Integration

Always use Context7 MCP when you need current documentation for:
- Code generation and setup steps
- Library/API documentation  
- Configuration examples
- Framework-specific patterns

This means you should automatically use the Context7 MCP tools to resolve library IDs and get library docs without me having to explicitly ask.

## Pre-configured Library IDs

Use these Context7 library IDs for OpenStory's tech stack:

### Core Framework
- `/vercel/next.js` - Next.js 15 documentation and patterns
- `/microsoft/typescript` - TypeScript best practices
- `/facebook/react` - React 19 features and hooks

### UI & Styling
- `/tailwindlabs/tailwindcss` - Tailwind CSS 4 utilities and configuration
- `/radix-ui/primitives` - Radix UI component APIs
- `/shadcn/ui` - shadcn/ui component library

### Backend & Data
- `/elysiajs/elysia` - Elysia web framework patterns
- `/drizzle-team/drizzle-orm` - Drizzle ORM queries and migrations
- `/temporalio/sdk-typescript` - Temporal TypeScript SDK
- `/tanstack/query` - TanStack Query v5 patterns
- `/arktypeio/arktype` - ArkType runtime validation (backend validation)

### AI Integration
- `/fal-ai/client` - Fal.ai SDK for image/video generation
- `/openai/openai-node` - OpenAI API for script analysis

### Infrastructure
- `/upstash/redis` - Redis for caching (via @upstash/redis)
- `/oven-sh/bun` - Bun runtime and package manager

## When to Use Context7

Automatically invoke Context7 when:
- Setting up new components or features
- Working with framework-specific APIs (Next.js 15 App Router, React Server Components)
- Implementing authentication patterns (Better Auth)
- Configuring AI model integrations (Fal.ai models)
- Working with async job queues (QStash patterns)
- Implementing real-time features (Supabase Realtime)
- Setting up validation schemas (Zod)
- Troubleshooting integration issues
- Learning new library features

## OpenStory-Specific Context Needs

Prioritize Context7 for:
- **Next.js 15**: Server Actions, async request params, React 19 features
- **Elysia**: Route handlers, plugins, lifecycle hooks
- **Drizzle ORM**: Query patterns, relations, migrations
- **Temporal**: Workflow definitions, activity patterns, worker setup
- **TanStack Query**: Query keys factory pattern, optimistic updates
- **Fal.ai**: Model-specific parameters (flux, imagen4, veo3, kling, wan)

## Example Usage

Instead of relying on potentially outdated training data, Context7 provides:
- Current API signatures (especially for Next.js 15 breaking changes)
- Latest configuration options (Tailwind 4, React 19)
- Up-to-date code examples (async params in App Router)
- Recent best practices (Server Components vs Client Components)
- Breaking changes and migrations (React 18→19, Next.js 14→15)

This ensures all generated code uses current, working patterns rather than deprecated or incorrect approaches.

---
> Source: [openstory-so/openstory](https://github.com/openstory-so/openstory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
