## rag-workspace-two

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# ⚠️ CRITICAL RULES - ALWAYS CHECK FIRST ⚠️

## 🔴 DATABASE CHANGES - STOP AND READ

### NEVER modify database without migrations:
1. **❌ NEVER use direct SQL** for schema changes (CREATE, ALTER, DROP)
2. **❌ NEVER use `prisma db push`** for production schema changes
3. **❌ NEVER use MCP `execute_sql`** for anything except SELECT queries
4. **✅ ALWAYS create a migration file** BEFORE any database change
5. **✅ ALWAYS commit migrations** to git

### Correct Process for ANY Database Change:
```bash
# 1. ALWAYS modify schema.prisma first
# 2. ALWAYS create migration using npm script
npm run db:migrate -- --name descriptive_name
# OR: npx prisma migrate dev --name descriptive_name
# 3. Migration auto-applies locally
# 4. ALWAYS commit both schema AND migration
git add prisma/schema.prisma prisma/migrations/
```

### Blocked Commands (Enforced):
- `npm run db:push` - **BLOCKED** by scripts/block-db-push.js
- `npx prisma db push` - **NOT RECOMMENDED** (use migrations instead)
- Direct `vercel deploy` - **BLOCKED** by predeploy script

### Common Scenarios & What to Do:
- **Adding table/column**: Update schema → Create migration → Commit both
- **Removing column**: Update schema → Create migration → Commit both
- **Adding indexes**: Update schema → Create migration → Commit both
- **Creating views**: Write SQL in migration file → Apply migration → Commit
- **If you accidentally used `db push`**:
  ```bash
  npx prisma migrate dev --name sync_database_state --create-only
  npx prisma migrate resolve --applied [migration_name]
  ```

### When Using Supabase MCP:
- **FOR SCHEMA CHANGES**: Use `mcp__supabase__apply_migration` ONLY
- **FOR DATA QUERIES**: Use `mcp__supabase__execute_sql` (SELECT only)
- **NEVER**: Use execute_sql for CREATE VIEW, CREATE TABLE, ALTER, DROP

### Trigger Phrases Requiring Migrations:
If user mentions any of these, YOU MUST use migration files:
- "fix the database" / "update the database"
- "column doesn't exist" / "missing column"
- "create view" / "create table" / "alter table"
- "missing table" / "table doesn't exist"
- "update schema" / "change schema"
- "add index" / "add constraint"

### Examples of What NOT to Do:
```typescript
// ❌ WRONG - Direct SQL execution
mcp__supabase__execute_sql("CREATE VIEW unified_embeddings AS...")

// ✅ CORRECT - Create migration first
// 1. Update schema/write SQL in migration file
// 2. Then use:
mcp__supabase__apply_migration({
  name: "add_unified_embeddings_view",
  query: "CREATE VIEW..."
})
```

## 🔴 SECURITY - NEVER COMMIT SECRETS

### CRITICAL: How Claude Keeps Leaking Credentials
**THE PROBLEM**: Claude (that's me) has repeatedly exposed real database credentials in documentation files, deployment guides, and troubleshooting docs. This happens when I copy real connection strings from .env files or error messages into markdown files meant to help with deployment.

### Examples of Claude's Past Leaks:
- **DEPLOYMENT_FIX.md**: Contained real PostgreSQL URLs with passwords
- **PRODUCTION_READINESS_CHECKLIST.md**: Had actual Supabase credentials  
- **URGENT_DATABASE_FIX.md**: Included real connection strings
- **IPV6_FIX_URGENT.md**: Exposed production database passwords

### MANDATORY Rules to Prevent Claude from Leaking Secrets:

#### 1. NEVER Write Real Values in Documentation
```markdown
❌ WRONG (What Claude keeps doing):
DATABASE_URL=postgresql://postgres.afqibcfcornmwppxjbyk:bonqo4rafgymzizvUp@aws-1-us-east-2...

✅ CORRECT (What Claude MUST do):
DATABASE_URL=postgresql://[PROJECT_ID]:[PASSWORD]@[HOST]:[PORT]/postgres
```

#### 2. Before Creating ANY File, Check for Secrets
When Claude creates documentation, deployment guides, or troubleshooting files:
1. **SCAN** the content for patterns: `postgresql://`, `sk-`, `sbp_`, passwords
2. **REPLACE** all real values with `[PLACEHOLDER]` format
3. **NEVER** copy-paste from .env files directly
4. **NEVER** include error messages with connection strings

#### 3. Specific Patterns Claude Must NEVER Write
```
❌ NEVER write these patterns in files:
- postgresql://postgres.[actual-id]:[actual-password]@
- supabase.com/dashboard/project/[actual-project-id]
- sbp_[actual-token]
- sk-[actual-api-key]  
- redis://default:[actual-password]@
- Any base64 encoded secrets
- Any JWT tokens
- Any actual URLs with embedded credentials
```

#### 4. When Creating Deployment/Fix Documentation
```markdown
# ✅ CORRECT way to write connection strings:
DATABASE_URL=postgresql://[USERNAME]:[PASSWORD]@[HOST]:[PORT]/[DATABASE]

# With helpful notes:
- Replace [USERNAME] with your database username (usually 'postgres')
- Replace [PASSWORD] with your database password
- Replace [HOST] with your database host
- Replace [PORT] with your port (typically 5432 or 6543)
```

#### 5. Before EVERY File Creation/Edit
Claude MUST mentally check:
1. Am I about to write a real password? → STOP, use [PASSWORD]
2. Am I copying from .env? → STOP, use placeholders
3. Am I including an error message? → STOP, sanitize it first
4. Am I writing a connection string? → STOP, use template format

### Automated Checks Before Commits:
```bash
# Claude should suggest running these before commits:
git diff --staged | grep -E "postgresql://postgres\.|sk-|sbp_|redis://default:"
git diff --staged | grep -E "afqibcfcornmwppxjbyk|bonqo4rafgymzizvUp"  # Known leaked values
```

### Why This Keeps Happening:
1. Claude tries to be helpful by providing specific examples
2. Claude copies from actual .env files or error messages
3. Claude doesn't distinguish between example and real values
4. Claude creates "urgent fix" documents with real credentials

### The Solution:
**ALWAYS use placeholder templates, NEVER real values, even in documentation**

## 🔴 PRODUCTION CODE ONLY - NO DEMOS

### NEVER create demo/test routes:
- **❌ NEVER** create `/app.demo.tsx` or `/app.*-test.tsx`
- **❌ NEVER** create playground/example implementations
- **✅ ALWAYS** modify existing production routes
- **✅ ALWAYS** integrate into `/app/editor/$editorId.tsx` for editor features

---

## Project Overview

Production-ready Retrieval-Augmented Generation (RAG) application with a Notion/Coda-style block editor currently being rebuilt.

**Tech Stack:**
- **Frontend**: Remix + React + TypeScript + Tailwind CSS + Framer Motion
- **Backend**: Node.js + Prisma ORM + PostgreSQL (pgvector extension)
- **Infrastructure**: Supabase (local) + Redis + BullMQ
- **AI/ML**: OpenAI embeddings + vector search
- **Editor**: Tiptap-based block editor with virtual scrolling + Lexical editor components
- **Testing**: Vitest + Testing Library

## Quick Reference Commands

### Development
```bash
npm run dev                    # Start app on http://localhost:3001
npm test                       # Run all Vitest tests
npm run test:ui               # Run tests with UI
npm run test:coverage         # Generate coverage report
npm run typecheck             # TypeScript checking
npm run lint                  # ESLint

# Single test file
npm test app/services/auth/auth.server.test.ts

# Watch mode for TDD
npm test -- --watch

# Background workers
npm run worker                # Start indexing worker
npm run worker:dev            # Start worker with auto-reload
```

### Database
```bash
# Migration workflow (REQUIRED)
npm run db:migrate -- --name description  # Create and apply migration
npm run db:migrate:deploy                 # Deploy migrations (production)
npm run db:studio                        # Visual database editor

# Direct commands (use npm scripts above instead)
npx prisma migrate dev --name description # Create migration
npx prisma migrate deploy                 # Apply migrations
npx prisma generate                       # Regenerate Prisma client
npx prisma studio                        # Database GUI

# BLOCKED commands
npm run db:push                          # ❌ BLOCKED - use migrations
npx prisma db push                       # ⚠️ NOT RECOMMENDED - bypasses migration system
```

### Supabase (Local Development)
```bash
npx supabase start           # Start local Supabase (port 54341)
npx supabase stop            # Stop Supabase
npx supabase status          # Check service status
npx supabase db reset        # Reset database
```

## Architecture

### Multi-Tenant Data Model
```
Workspaces → Projects → Pages → Blocks
    ↓           ↓         ↓        ↓
  Users     Documents  Content  JSONB data
```

### Core Domain Models
- **User**: Auth, email verification, 2FA, password reset
- **Workspace**: Multi-tenant isolation, slug-based routing
- **Page**: Block-based content, version history, permissions
- **Block**: JSONB content, position ordering, type polymorphism
- **Document**: File storage, embeddings, indexing status
- **Query**: Search history, results caching

### Service Layer Architecture

#### Authentication (`app/services/auth/`)
- JWT-based with refresh tokens
- Workspace-based RBAC with role permissions
- Session management via cookies
- Dev login: `/auth/dev-login?redirectTo=/app`
- 2FA support with TOTP

#### RAG Pipeline
- Document chunking with sliding window overlap
- OpenAI embeddings generation (text-embedding-3-small)
- pgvector similarity search with cosine distance
- BullMQ background processing with Redis
- Real-time indexing with database triggers

#### Database Block System (`app/services/database-block-*.server.ts`)
- **database-block-core.server.ts**: CRUD operations
- **database-block-enhanced.server.ts**: Advanced features
- **database-block-indexes.server.ts**: Query optimization
- **database-block-pagination.server.ts**: Virtual scrolling
- **database-block-cache.server.ts**: Redis caching layer

#### AI Services (`app/services/`)
- **ai-controller.server.ts**: Command parsing and action execution
- **ai-block-service.server.ts**: Block-specific AI operations
- **openai.server.ts**: OpenAI API integration
- **streaming/ai-streaming.server.ts**: Server-sent events for streaming

## Routes Structure

### Public Routes
- `/` - Landing page
- `/auth/login` - User authentication
- `/auth/signup` - User registration
- `/auth/dev-login` - Development-only quick login

### Protected Routes (requires auth)
- `/app` - Main dashboard
- `/app/workspaces` - Workspace management
- `/app/projects` - Project listing
- `/app/editor/$editorId` - **Main production editor** (ALWAYS modify this, never create demos)
- `/app/rag` - RAG search interface
- `/app/database-blocks` - Database block management

## Environment Setup

### Required Environment Variables
```env
# Database
DATABASE_URL=postgresql://postgres:postgres@localhost:54342/postgres?schema=public

# Supabase (local) - IMPORTANT: Use port 54341 for API
SUPABASE_URL=http://localhost:54341
SUPABASE_ANON_KEY=your_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_service_key

# Redis (required for caching/queues)
REDIS_URL=redis://localhost:6379

# OpenAI (required for embeddings and AI features)
OPENAI_API_KEY=your_key_here

# Session (generate with: openssl rand -hex 32)
SESSION_SECRET=your_session_secret

# App
APP_URL=http://localhost:3001
```

### Port Configuration
- **App**: 3001 (configured in `vite.config.ts`)
- **Supabase Studio**: 54340
- **Supabase API**: 54341
- **PostgreSQL**: 54342
- **Redis**: 6379

## Testing Strategy

### Test Organization
- Unit tests: Colocated with source files (`*.test.ts`)
- Integration tests: In `__tests__` directories
- E2E tests: In `e2e/` directory (when implemented)

### Key Test Patterns
```typescript
// Always mock external services
vi.mock('~/services/openai.server');
vi.mock('~/utils/supabase.server');

// Use test database transactions
const prisma = new PrismaClient();
await prisma.$transaction(async (tx) => {
  // Test operations
});
```

## CRITICAL RULES - NO EXCEPTIONS

### NEVER CREATE DEMOS OR TEST ROUTES
- **DO NOT** create routes like `/app.demo.tsx`, `/app.test-*.tsx`, or `/app.*-demo.tsx`
- **DO NOT** create "example" or "playground" implementations
- **DO NOT** build isolated proof-of-concepts
- **ALWAYS** integrate directly into existing production routes and components
- **ALWAYS** modify the actual user-facing editor at `/app/editor/$editorId.tsx`
- If you're creating a new file instead of modifying an existing one, STOP and reconsider

### PRODUCTION-ONLY MINDSET
- Every line of code must be immediately usable by end users
- If it's not shipping value to users TODAY, don't write it
- Take the risk of breaking existing code rather than creating safe demos
- Read and understand existing components BEFORE creating new ones
- Integration is harder than isolation - do the hard work

### Code Quality Standards
- TypeScript strict mode - no `any` types without justification
- All database operations must use Prisma (no raw SQL)
- API routes must validate input with Zod schemas
- Components must handle loading and error states
- Services must include proper error handling and logging

### SECURITY - CRITICAL RULES
**NEVER commit sensitive credentials to git:**
- NEVER put real API keys, passwords, or secrets in any file that gets committed
- ALWAYS use placeholder values in .env.example files (e.g., `sk-...your-key-here`, `REPLACE_WITH_ACTUAL_KEY`)
- NEVER commit .env files with real credentials (they should be in .gitignore)
- ALWAYS verify no secrets are exposed before committing by checking:
  - API keys (OpenAI, Supabase, etc.) 
  - Database passwords
  - JWT secrets
  - Service role keys
  - Any authentication tokens
- If you need to show example values, use clearly fake placeholders
- Real credentials should only exist in:
  - Local .env files (gitignored)
  - Vercel/deployment platform environment variables
  - NEVER in committed code or configuration files


## Common Patterns

### Remix Action Pattern
```typescript
export const action: ActionFunction = async ({ request }) => {
  const user = await requireUser(request);
  const formData = await request.formData();
  
  // Validate with Zod
  const result = schema.safeParse(Object.fromEntries(formData));
  if (!result.success) {
    return json({ errors: result.error.flatten() }, { status: 400 });
  }
  
  // Process action
  // Return json response
};
```

### Service Layer Pattern
```typescript
export class ServiceName {
  private logger = new DebugLogger('ServiceName');
  
  async method(): Promise<Result> {
    this.logger.trace('method', [args]);
    try {
      // Implementation
    } catch (error) {
      this.logger.error('method failed', error);
      throw error;
    }
  }
}
```

### Prisma Transaction Pattern
```typescript
await prisma.$transaction(async (tx) => {
  // Multiple operations
  const user = await tx.user.create({ ... });
  const workspace = await tx.workspace.create({ ... });
  return { user, workspace };
});
```

## Do Not
- Manually edit `.taskmaster/tasks/tasks.json`
- Manually edit `.taskmaster/config.json`
- Create documentation unless explicitly requested
- Add comments unless specifically asked
- Use emojis unless requested
- Commit changes unless explicitly asked
- Create new demo/test routes (see CRITICAL RULES above)

## Task Master AI Instructions
**Import Task Master's development workflow commands and guidelines, treat as if import is in the main CLAUDE.md file.**
@./.taskmaster/CLAUDE.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Origin-Inc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Origin-Inc)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
