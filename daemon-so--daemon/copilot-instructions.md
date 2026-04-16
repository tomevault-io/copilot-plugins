## daemon

> > **⚠️ Maintenance Note for Future Agents**: This file contains critical patterns and conventions discovered through codebase exploration. When you add new important patterns, utilities, or architectural decisions, please update this file to help future context gathering. Focus on patterns that would save time if documented upfront (e.g., utility functions, common code patterns, configuration details).

# Daemon Project Rules

> **⚠️ Maintenance Note for Future Agents**: This file contains critical patterns and conventions discovered through codebase exploration. When you add new important patterns, utilities, or architectural decisions, please update this file to help future context gathering. Focus on patterns that would save time if documented upfront (e.g., utility functions, common code patterns, configuration details).

## Project Context

You are working on Daemon, an AI agent deployment platform. The project uses:

- **Frontend**: Next.js 16 (App Router), React 19, TypeScript, Tailwind CSS
- **Backend**: Supabase (PostgreSQL, Auth, RLS)
- **Authentication**: OAuth-only (Google, GitHub)
- **Monorepo**: Turborepo with apps/web, packages/cli, packages/db
- **Styling**: Tailwind + shadcn/ui components
- **Database**: PostgreSQL with Row Level Security (RLS)

## Code Style & Conventions

### TypeScript

- Use TypeScript for all code
- Prefer `async/await` over promises
- Use type inference where possible
- Avoid `any` types - use `unknown` if needed

### React & Next.js

- Use Server Components by default
- Only use "use client" when necessary (state, effects, browser APIs)
- Prefer server-side data fetching with `async` functions
- Use Next.js App Router conventions (not Pages Router)

### Code Organization

- Keep files small and focused (under 300 lines)
- Return early to avoid else clauses
- Prefer composition over inheritance
- Use named exports over default exports

### Naming Conventions

- Files: `kebab-case.tsx`
- Components: `PascalCase`
- Functions: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE`
- Database tables: `snake_case`

## Supabase & Database

### Authentication

- OAuth-only (Google, GitHub) - no email/password
- Use `@/lib/supabase/server` for Server Components
- Use `@/lib/supabase/client` for Client Components
- Always check auth before protected operations
- User profiles auto-created via database trigger

### Database Access

- All tables have Row Level Security (RLS) enabled
- Use team-based access control via `team_members` table
- Never bypass RLS in application code
- Service role key only for admin operations

### Queries

- Prefer server-side queries with proper RLS
- Use `getUser()` helper for auth checks
- Handle errors gracefully
- Log database errors for debugging

### Supabase Client Patterns

- **Server Components**: Use `createClient()` from `@/lib/supabase/server` (auth-aware, uses cookies)
- **Client Components**: Use `createClient()` from `@/lib/supabase/client` (browser client)
- **Admin Operations**: Use `createSupabaseServer()` from `@/lib/supabase` (service role, bypasses RLS)
- **Middleware**: Session refresh handled in `@/lib/supabase/middleware.ts` - call `updateSession()` in Next.js middleware
- **Helpers**: `getUser()`, `getUserProfile()`, `getSession()` available from `@/lib/supabase/server`

## UI & Styling

### Design System

- Use shadcn/ui components from `@/components/ui`
- Follow existing component patterns
- Dark mode first, light mode compatible
- Use Tailwind utility classes, avoid custom CSS

### Tailwind CSS Gotchas

- **Opacity modifiers with CSS variables**: To robustly support opacity modifiers (e.g., `border-border/80`) with CSS variables, use a helper function in `tailwind.config.ts` instead of string placeholders. This avoids parsing ambiguities.
  ```ts
  function withOpacity(variableName: string) {
    return ({ opacityValue }: { opacityValue?: string }) => {
      if (opacityValue !== undefined) {
        return `hsl(var(${variableName}), ${opacityValue})`;
      }
      return `hsl(var(${variableName}))`;
    };
  }
  // usage: colors: { border: withOpacity("--border") }
  ```

### Components

- Keep components simple and reusable
- Use TypeScript interfaces for props
- Handle loading and error states
- Optimize for performance (use `memo` sparingly)

### UI Patterns

- **No slide-out panels (Sheet)** - Use pages or modals (Dialog) instead
- Use `Dialog` from shadcn/ui for quick actions, confirmations, or forms that don't need a full page
- Use dedicated pages for complex content that benefits from URL routing
- Mobile navigation should use full-screen pages, not slide-outs

### Accessibility

- Use semantic HTML
- Include proper ARIA labels
- Ensure keyboard navigation works
- Test with screen readers

## Error Handling

### Frontend

- Show user-friendly error messages
- Log errors to console for debugging
- Handle loading states properly
- Provide fallback UI when needed

### Backend

- Return appropriate HTTP status codes
- Include helpful error messages
- Never expose sensitive data in errors
- Log errors with context

## Security

### Authentication & Authorization

- Always verify user authentication server-side
- Check team membership for multi-tenant features
- Use RLS policies for database access
- Validate all user input

### Data Protection

- Never expose service role keys client-side
- Use environment variables for secrets
- Sanitize user input
- Follow OWASP guidelines

## Testing

### Manual Testing

- Test OAuth flow with both providers
- Verify RLS policies work correctly
- Check protected routes redirect properly
- Test error states and edge cases

### Code Quality

- Run linter before committing
- Fix TypeScript errors immediately
- Keep bundle size reasonable
- Optimize images and assets

## Documentation

### Code Comments

- Document complex logic and business rules
- Explain "why" not "what" in comments
- Keep comments up to date with code
- Use JSDoc for public APIs

### File Organization

- Related files stay together
- Clear folder structure
- Logical component hierarchy
- Easy to navigate

## Performance

### Optimization

- Use Server Components for static content
- Lazy load heavy components
- Optimize images with Next.js Image
- Minimize client-side JavaScript

### Database

- Index frequently queried columns
- Avoid N+1 queries
- Use connection pooling
- Monitor query performance

## Development Workflow

### Making Changes

- Be surgical - make minimal necessary changes
- Prefer small, focused PRs
- Test changes locally before committing
- Update documentation if needed

### Changelog Updates

- **Mandatory Update**: Anytime a new feature is shipped, you must add a changelog entry.
- **Location**: `apps/web/content/changelog/`
- **File Naming**: `YYYY-MM-DD-descriptive-slug.md` (e.g., `2025-11-23-new-feature.md`)
- **Template**:

  ```yaml
  ---
  title: "Title of Change"
  date: "YYYY-MM-DD"
  version: "0.0.0" # Optional: bump version if applicable
  tags: ["Feature", "Fix", "Improvement"]
  ---
  - Description of the changes...
  ```

### When Stuck

1. Check Supabase logs (localhost:54323)
2. Check browser console for errors
3. Verify environment variables are set
4. Review RLS policies in database
5. Check middleware is running

## Environment Variables & Configuration

### Environment Variable Access

- **Always use `@/lib/env`** - Never access `process.env` directly in application code
- Next.js config loads `.env` files from repo root (not just `apps/web`)
- Environment variables are cached and normalized (trimmed) automatically
- Use `env.supabaseUrl()`, `env.supabaseAnonKey()`, etc. from the `env` object

### Required Environment Variables

- `NEXT_PUBLIC_SUPABASE_URL` - Supabase project URL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` - Supabase anonymous key (public, safe for client)
- `SUPABASE_SERVICE_ROLE_KEY` - Service role key (secret, server-only)
- `AI_GATEWAY_API_KEY` - Vercel AI Gateway API key (secret, server-only)
- `AI_GATEWAY_APP_ID` - Vercel AI Gateway app attribution ID

### Optional Environment Variables

- `EXA_API_KEY` - Required for web search tool (`web.search`)
- `WORKFLOW_RUN_ENDPOINT` - Custom workflow execution endpoint
- `AI_GATEWAY_BASE_URL` - Defaults to `https://ai-gateway.vercel.sh/v1` if not set
- `NEXT_PUBLIC_DAEMON_BASE_URL` - Defaults to `http://localhost:3000` if not set

### Environment Variable Patterns

```typescript
import { env } from "@/lib/env";

// Required (throws if missing)
const url = env.supabaseUrl();

// Optional with fallback
const baseUrl = env.daemonBaseUrl(); // Has default fallback

// Optional (returns undefined if missing)
const endpoint = env.workflowRunEndpoint();
```

### Common Patterns

#### Protected Server Component

```typescript
import { getUser } from "@/lib/supabase/server";
import { redirect } from "next/navigation";

export default async function ProtectedPage() {
  const user = await getUser();
  if (!user) redirect("/");

  // Page content
}
```

#### OAuth Sign In (Client)

```typescript
const supabase = createClient();
await supabase.auth.signInWithOAuth({
  provider: "google",
  options: {
    redirectTo: `${window.location.origin}/api/auth/callback`,
  },
});
```

#### Database Query with RLS

```typescript
const supabase = await createClient();
const { data, error } = await supabase.from("agents").select("*").eq("team_id", teamId);
```

#### AI Model Calls (Vercel AI Gateway)

```typescript
import { createOpenAI } from "@ai-sdk/openai";
import { generateText } from "ai";
import {
  DEFAULT_MODEL,
  DEFAULT_AI_GATEWAY_BASE_URL,
  AI_GATEWAY_APP_HEADERS,
} from "@/lib/constants/ai";

const apiKey = process.env.AI_GATEWAY_API_KEY;
if (!apiKey) {
  throw new Error("AI Gateway not configured");
}

const openai = createOpenAI({
  apiKey,
  baseURL: DEFAULT_AI_GATEWAY_BASE_URL,
  headers: AI_GATEWAY_APP_HEADERS,
});

const result = await generateText({
  model: openai(DEFAULT_MODEL),
  prompt: "Your prompt here",
});
```

#### API Route with Token Authentication

```typescript
import { NextRequest, NextResponse } from "next/server";
import { verifyApiToken } from "@/lib/api-tokens";
import { z } from "zod";

export async function POST(request: NextRequest) {
  // Extract Bearer token
  const authHeader = request.headers.get("authorization");
  const apiKey = authHeader?.replace(/^Bearer\s+/i, "").trim() || null;
  const token = await verifyApiToken(apiKey);

  if (!token) {
    return NextResponse.json(
      {
        error: {
          message: "Invalid API key",
          type: "invalid_request_error",
          code: "invalid_api_key",
        },
      },
      { status: 401 },
    );
  }

  // Validate request body with Zod
  const body = await request.json();
  const parsed = requestSchema.safeParse(body);

  if (!parsed.success) {
    return NextResponse.json(
      {
        error: {
          message: "Invalid request",
          type: "invalid_request_error",
          code: "invalid_request",
          details: parsed.error.issues,
        },
      },
      { status: 400 },
    );
  }

  // Use token.teamId for team-scoped operations
  // ...
}
```

#### Utility Functions

```typescript
// ClassName merging (combines clsx + tailwind-merge)
import { cn } from "@/lib/utils";
<div className={cn("base-class", condition && "conditional-class")} />

// Structured logging (not console.log)
import { logger } from "@/lib/logger";
logger.info("Message", { context: { userId, action: "create" } });
logger.error("Error occurred", { error, context });

// Environment variables (not process.env directly)
import { env } from "@/lib/env";
const url = env.supabaseUrl();
const key = env.supabaseAnonKey();
```

## Project-Specific Rules

### Multi-Tenancy

- All user data belongs to a team
- Users can be members of multiple teams
- Team owners can manage team resources
- RLS enforces team boundaries

### Agent Deployment

- Agents defined in daemon.json
- CLI deploys agents to platform
- Agents run autonomously in cloud
- Runs tracked in database

### Architecture Decisions

- OAuth-only for security and UX
- RLS for data isolation
- Server Components for performance
- Tailwind for rapid UI development
- **Vercel AI Gateway for all AI model calls** - Never use direct OpenAI API keys (`OPENAI_API_KEY`). Always use `AI_GATEWAY_API_KEY` with `DEFAULT_AI_GATEWAY_BASE_URL` and `AI_GATEWAY_APP_HEADERS` from `@/lib/constants/ai`.

## Workflow & Agent Execution System

### Workflow Architecture

- **Workflow-based execution**: Agents run via DAG (plan → execute) workflow system
- **DAG Planner**: Generates task DAG from agent configuration and user input
- **DAG Runtime**: Executes tasks in parallel when dependencies are met
- **Skills System**: Agents use skills (chat, action, etc.) selected by router step
- **Event Streaming**: Workflow execution uses async generators (`async function*`) for streaming events

### Default Tools

- Use `withDefaultWorkflowTools()` from `@/lib/workflows/tools/utils` to automatically add:
- Tool surface is defined by the agent’s enabled skills/tools. `withDefaultWorkflowTools()` only trims and deduplicates the provided list—no implicit defaults are injected today.
- If the runtime ever needs a mandatory tool, document it alongside `MANDATORY_WORKFLOW_TOOLS` in `apps/web/lib/workflows/tools/utils.ts` and update the planner/README accordingly.

### Workflow Execution Pattern

```typescript
import { executeRunWorkflow } from "@/lib/workflows/execute-run";
import { withDefaultWorkflowTools } from "@/lib/workflows/tools/utils";

const agentTools = withDefaultWorkflowTools(providedAgentTools);
const workflowInput = {
  runId,
  conversationId,
  messages,
  agentPrompt,
  enableThinking: true,
  agentTools,
  agentId,
  teamId,
};

// Workflow returns async generator of events
for await (const event of executeRunWorkflow(workflowInput)) {
  // Stream events to client
  yield event;
}
```

### Skills System

- Skills provide different "modes" or capabilities for agents
- Router step selects which skill to execute based on user input
- Each skill has: instructions, tool allowlist, tone/style guidelines
- Skills defined in `@/lib/skills/default-skills.ts`
- Use `routeToSkill()` to select skill, `prepareSkillExecution()` to prepare execution

## Tool System Architecture

### Tool Definition

Tools implement the `RegisteredTool` interface:

```typescript
import type { RegisteredTool } from "@/lib/tools/types";
import { z } from "zod";

const myTool: RegisteredTool = {
  title: "Human-readable title",
  toolId: "tool.identifier", // Use dot notation
  description: "What the tool does (for LLM)",
  parameters: z.object({
    // Zod schema for validation
  }),
  async invoke({ agentId, teamId, args }) {
    // Tool execution logic
    return result; // string or Record<string, unknown>
  },
};
```

### Tool Registration

- Tools registered in `@/lib/tools/registry.ts`
- Use `getRegisteredTool(toolId)` to resolve a tool by ID
- Use `listRegisteredTools()` to get all registered tool IDs
- Tool IDs use dot notation: `web.search`, `agent_data.create_table`, `memory.save`

### Tool Resolution

- Use `resolveToolSet()` from `@/lib/tools/resolve-tool-set` to resolve tool names to definitions
- Tools can be referenced by full ID (`web.search`) or short name (`github` for GitHub tools)
- Tool registry includes: base tools (slack, webhook), GitHub tools, agent_data tools, memory tools

### Tool Locations

- Tool definitions: `apps/web/lib/tools/` - each tool in its own file
- Tool registry: `apps/web/lib/tools/registry.ts`
- Tool types: `apps/web/lib/tools/types.ts`

## AI Model Configuration

### Model Constants

- Use `DEFAULT_MODEL` from `@/lib/constants/ai` (currently `"gpt-oss-120b"`)
- Use `REASONING_MODEL` for reasoning tasks (currently `"gpt-oss-120b"`)
- Use `shouldUseReasoning(enableThinking)` to determine if reasoning model should be used
- Reasoning models: `REASONING_MODELS` enum includes GPT-OSS variants and o1 series

### Gateway Configuration

- Always use Vercel AI Gateway (never direct OpenAI API)
- Use `DEFAULT_AI_GATEWAY_BASE_URL` from `@/lib/constants/ai`
- Use `AI_GATEWAY_APP_HEADERS` for app attribution (includes `x-vercel-ai-gateway-app-id`)
- Gateway requires `AI_GATEWAY_API_KEY` environment variable

### Model Usage Pattern

```typescript
import { createOpenAI } from "@ai-sdk/openai";
import { generateText } from "ai";
import {
  DEFAULT_MODEL,
  REASONING_MODEL,
  DEFAULT_AI_GATEWAY_BASE_URL,
  AI_GATEWAY_APP_HEADERS,
  shouldUseReasoning,
} from "@/lib/constants/ai";

const modelToUse = shouldUseReasoning(enableThinking) ? REASONING_MODEL : DEFAULT_MODEL;
const openai = createOpenAI({
  apiKey: process.env.AI_GATEWAY_API_KEY!,
  baseURL: DEFAULT_AI_GATEWAY_BASE_URL,
  headers: AI_GATEWAY_APP_HEADERS,
});

const result = await generateText({
  model: openai(modelToUse),
  prompt: "Your prompt",
});
```

## Type Definitions & Interfaces

### Common Type Locations

- **Tool types**: `@/lib/tools/types.ts` - `RegisteredTool`, `ResolveToolSetOptions`
- **Workflow types**: `@/lib/workflows/event-types.ts` - `WorkflowEvent`, `WorkflowTaskStatus`
- **Agent data types**: `@/lib/agent-data/types.ts` - `AgentDataCommand`, `AgentDataResult`, `ColumnDef`
- **Skill types**: `@/lib/skills/types.ts` - `AgentSkill`, `SkillRouteResult`
- **DAG types**: `@/lib/workflows/dag/types.ts` - `PlannerTaskDefinition`, `ConversationMessage`

### Type Usage Patterns

- Import types from their respective modules
- Use Zod schemas for runtime validation, TypeScript types for compile-time checking
- Common pattern: `export type TypeName = z.infer<typeof zodSchema>`

## Don't Do This

- ❌ Don't add email/password authentication
- ❌ Don't bypass RLS policies
- ❌ Don't use Client Components unnecessarily
- ❌ Don't expose secrets in client code
- ❌ Don't make database queries without error handling
- ❌ Don't skip TypeScript types
- ❌ Don't ignore linter warnings
- ❌ Don't make unrequested changes
- ❌ **Don't use `OPENAI_API_KEY`** - Always use `AI_GATEWAY_API_KEY` with Vercel AI Gateway. Never call OpenAI API directly.
- ❌ Don't access `process.env` directly - Use `@/lib/env` instead
- ❌ Don't use `console.log` for logging - Use `logger` from `@/lib/logger`
- ❌ Don't create tools without registering them in `@/lib/tools/registry.ts`

## Documentation & Notes

### Documentation Location

All project documentation is in `docs/`:

- `docs/BRAND_ASSETS.md` - Brand identity, assets, and visual design philosophy
- `docs/CODEX_CLOUD.md` - Codex Cloud setup guide
- `docs/PRODUCTION_MIGRATIONS.md` - Database migration workflow
- `docs/STATSIG_INTEGRATION.md` - Feature flags integration
- `docs/SPEC_*.md` - Design specifications (e.g., `SPEC_agent-scheduling.md`, `SPEC_workflow-dag.md`)
- `docs/PLAN_*.md` - Planning documents (e.g., `PLAN_launch.md`, `PLAN_dmn-cli-tui.md`)

### Documentation Rules

- **All documentation goes in `docs/`** - Never create .md files in repo root (except README.md)
- Use `SPEC_` prefix for design specifications
- Use `PLAN_` prefix for planning documents
- Keep docs organized and updated
- Reference docs in code comments when relevant

Refer to these for implementation details and setup instructions.

## Database Migrations

### Migration Strategy

- **Always use migrations for schema changes** - Never provide raw SQL snippets to run manually
- Create idempotent migrations that can be run multiple times safely
- Use `IF NOT EXISTS` and conditional checks in all DDL statements
- Migrations live in `supabase/migrations/`

### Migration Naming

- Format: `YYYYMMDDHHMMSS_description.sql` (14 digits, NO underscore between date and time)
- Use today's date + time: `YYYYMMDDHHMMSS` (e.g., `20251121133114`)
- Get the current timestamp with `date "+%Y%m%d%H%M%S"`
- Keep descriptions short and clear (e.g., `enable_rls.sql`, `fix_vector_operator.sql`)
- **IMPORTANT**: Supabase requires 14-digit format without underscore. Format `20251120_095532` is WRONG, use `20251120095532` instead

### Migration Content

- Always include header comment explaining purpose
- Use `do $$ ... end $$;` blocks for conditional logic
- Check for existence before creating tables, indexes, policies, functions
- Enable RLS explicitly where needed
- Include rollback notes if migration is complex

### Example Migration Structure

```sql
-- Purpose: Create team_members table with RLS
-- Safe to run multiple times (idempotent)

-- Create table if not exists
do $$
begin
  if not exists (
    select 1 from pg_tables
    where schemaname = 'public' and tablename = 'team_members'
  ) then
    create table public.team_members (
      id uuid primary key default gen_random_uuid(),
      -- ... columns
    );
  end if;
end $$;

-- Enable RLS
alter table public.team_members enable row level security;

-- Add policies conditionally
do $$
begin
  if not exists (
    select 1 from pg_policies
    where schemaname = 'public'
      and tablename = 'team_members'
      and policyname = 'Users can view own team memberships'
  ) then
    create policy "Users can view own team memberships"
      on public.team_members for select
      using (auth.uid() = user_id);
  end if;
end $$;
```

### Applying Migrations

- Use Supabase CLI: `supabase db push` (local) or apply via SQL Editor (hosted)
- Test migrations locally first with `supabase start`
- Migrations are applied in filename order
- Never edit applied migrations - create new ones for changes
- **See `supabase/migrations/MIGRATION_WORKFLOW.md` for troubleshooting migration drift, function signature conflicts, and connection issues**
- When changing function parameter names, always use `DROP FUNCTION IF EXISTS ... CASCADE` before `CREATE FUNCTION` (PostgreSQL doesn't allow renaming parameters with `CREATE OR REPLACE`)

## Intentional Exceptions

### Console Usage Exceptions

The following files intentionally use `console.*` instead of `logger`:

**Client Components** (logger is server-only):

- `components/chat/conversation-view.tsx` - Debug logging for streaming
- `components/agents/agent-limit-paywall.tsx` - Error reporting
- `components/chat/integration-prompt.tsx` - Error reporting
- `components/calendar/calendar-dashboard.tsx` - Error reporting
- `components/ai-elements/prompt-input.tsx` - Speech recognition errors
- `components/docs/*.tsx` - Documentation components
- `components/onboarding/onboarding-wizard.tsx` - Error reporting
- `hooks/use-realtime-*.ts` - Realtime hooks
- `app/app-layout.tsx` - Development warnings

**Edge Runtime** (logger adds overhead):

- `app/api/og/route.tsx` - OG image generation
- `app/api/og-alt/route.tsx` - OG image generation (alt)

**Logger Implementation**:

- `lib/logger.ts` - Uses console internally (intentional)

### Type Suppression Exceptions

The following patterns require type assertions due to dynamic data:

**Dynamic SQL operations** (`lib/agent-data/`):

- Supabase RPC functions use `any` for JSONB parameters
- This is validated at runtime by Zod schemas

**Tailwind config** (`tailwind.config.ts`):

- `withOpacity` helper uses `any` return type (Tailwind requirement)

## Testing Conventions

### Test File Location

- Place tests next to source: `lib/foo/bar.ts` → `lib/foo/bar.test.ts`
- Use `*.test.ts` for unit tests, `*.test.tsx` for component tests

### Test Templates

Templates available in `docs/templates/`:

- `api-route.test.template.ts` - API route testing patterns
- `hook.test.template.tsx` - Hook testing patterns
- `component.test.template.tsx` - Component testing patterns

### Running Tests

```bash
# Run all tests
npm run test --workspace @daemon/web

# Run specific test file
npm run test --workspace @daemon/web -- lib/skills/registry.test.ts

# Watch mode
npm run test:watch --workspace @daemon/web
```

### Test Patterns

- Mock Supabase with `vi.mock("@/lib/supabase/server")`
- Use `vi.useFakeTimers()` for time-dependent tests
- Test error cases, not just happy paths

## API Error Response Patterns

### Standard Error Format

Use `ApiErrors` from `@/lib/api-errors.ts`:

```typescript
import { ApiErrors } from "@/lib/api-errors";

// In route handlers:
if (!user) return ApiErrors.unauthorized();
if (!resource) return ApiErrors.notFound("Resource");
if (!valid) return ApiErrors.badRequest("Invalid input", zodErrors);
```

### Error Response Structure

```json
{
  "error": {
    "message": "Human-readable message",
    "type": "api_error | validation_error | auth_error",
    "code": "machine_readable_code",
    "details": {} // Optional, for validation errors
  }
}
```

### Common Error Codes

| Code             | Status | Usage                    |
| ---------------- | ------ | ------------------------ |
| `unauthorized`   | 401    | Missing or invalid auth  |
| `forbidden`      | 403    | Insufficient permissions |
| `not_found`      | 404    | Resource doesn't exist   |
| `bad_request`    | 400    | Invalid input            |
| `conflict`       | 409    | State conflict           |
| `rate_limited`   | 429    | Too many requests        |
| `internal_error` | 500    | Server error             |

### API Route Template

See `docs/templates/api-route.template.ts` for the canonical route structure.

## Architecture Decision Records

Key architectural decisions are documented in `docs/ADR_*.md`:

- `ADR_001_oauth-only-auth.md` - Why OAuth-only (no email/password)
- `ADR_002_skill-based-tools.md` - Tool organization philosophy
- `ADR_003_dag-workflow.md` - DAG-based execution model
- `ADR_004_supabase-client-patterns.md` - When to use which client

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/daemon-so) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
