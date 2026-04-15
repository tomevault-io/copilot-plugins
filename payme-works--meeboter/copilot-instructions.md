## meeboter

> This skill guides collaborative design through:

# Agent Rules

## Brainstorming Before Implementation (CRITICAL)

**ALWAYS USE THE SUPERPOWERS BRAINSTORMING SKILL BEFORE STARTING ANY TASK**

Before implementing any feature, fix, or change, you MUST use the `superpowers:brainstorming` skill:

```
Skill: superpowers:brainstorming
```

This skill guides collaborative design through:

1. **Understanding the idea** - Ask questions one at a time (prefer multiple choice) to refine requirements
2. **Exploring approaches** - Propose 2-3 different approaches with trade-offs and your recommendation
3. **Presenting the design** - Break design into 200-300 word sections, validating each before continuing
4. **Documentation** - Write validated design to `docs/plans/YYYY-MM-DD-<topic>-design.md`

**Key principles from the skill:**
- One question at a time, don't overwhelm
- Multiple choice questions preferred when possible
- YAGNI ruthlessly, remove unnecessary features
- Always explore 2-3 alternatives before settling
- Incremental validation of each design section

This applies to ALL tasks including: new features, bug fixes, refactoring, configuration changes, database migrations, and any code modifications.

## Rule Organization (CRITICAL)

**Each rule MUST be in its own dedicated file in `/rules`** - Never add rules directly to CLAUDE.md. This file only contains brief references to rules.

- **Create dedicated rule files** - Each topic gets its own file in `/rules` (e.g., `rules/URL_STATE.md`, `rules/API_PATTERNS.md`)
- **Brief references only in CLAUDE.md** - Only add a one-line reference in the Quick Reference table
- **Library-specific rules go in `/rules/libraries/`** - E.g., `rules/libraries/NUQS.md`

```bash
# ✅ CORRECT: Rule in dedicated file
rules/URL_STATE.md          # Contains full URL state patterns
CLAUDE.md                   # Just references: "URL_STATE.md | nuqs, search params, sort format"

# ❌ WRONG: Rule directly in CLAUDE.md
CLAUDE.md                   # Contains full URL state patterns inline
```

## Rule Lookup Protocol (MANDATORY)

**BEFORE starting ANY task, you MUST identify and read the relevant rule files.** This is not optional. Failing to consult rules leads to inconsistent code and rework.

### How to Use This Protocol

1. **Identify task type** from the trigger list below
2. **Read the referenced rule file(s)** using the Read tool BEFORE writing any code
3. **Apply the patterns** exactly as documented
4. **When in doubt, search rules/** - Use `Glob` or `Grep` to find relevant rules

### Task-Based Rule Lookup

#### When Working on AUTHENTICATION or API KEYS
**Triggers**: Auth flows, sessions, API keys, Better-Auth, OAuth, sign-in/sign-up
**MUST READ**:
- [rules/AUTHENTICATION.md](rules/AUTHENTICATION.md) - Better-Auth SDK, API key patterns, auth hooks

**Search for**: `Grep pattern="auth|api.key|session|oauth" path="rules/"`

#### When Working on REACT COMPONENTS or UI
**Triggers**: Creating components, dialogs, sheets, tables, forms, styling
**MUST READ**:
- [rules/COMPONENT_PATTERNS.md](rules/COMPONENT_PATTERNS.md) - Component organization, Sheet vs Dialog, z-index, tables
- [rules/CODE_STYLE.md](rules/CODE_STYLE.md) - JSX conventions, composition patterns, React keys
- [rules/libraries/NEXTJS.md](rules/libraries/NEXTJS.md) - "use client"/"use server" directives, component boundaries

**Search for**: `Grep pattern="component|dialog|sheet|table|jsx" path="rules/"`

#### When Working on DATA TABLES (TanStack React Table)
**Triggers**: Creating tables, table columns, table meta, row selection, filtering, useReactTable
**MUST READ**:
- [rules/libraries/REACT_TABLE.md](rules/libraries/REACT_TABLE.md) - Table meta types, column definitions, custom filters, row selection

**Search for**: `Grep pattern="useReactTable|TableMeta|ColumnDef|react-table" path="rules/"`

#### When Working on NAMING (Interfaces, Variables, Files)
**Triggers**: Naming interfaces, types, booleans, enums, files, components
**MUST READ**:
- [rules/NAMING_CONVENTIONS.md](rules/NAMING_CONVENTIONS.md) - Interface naming, boolean prefixes, dialog naming, **no Details/Info/Data suffixes**

**Search for**: `Grep pattern="naming|interface|boolean|enum" path="rules/"`

#### When Working on TESTS
**Triggers**: Writing E2E tests, unit tests, tRPC tests, browser tests
**MUST READ**:
- [rules/TESTING.md](rules/TESTING.md) - E2E patterns, Better-Auth testing, Playwright MCP

**Search for**: `Grep pattern="test|e2e|playwright|mock" path="rules/"`

#### When Working on ZOD SCHEMAS
**Triggers**: Creating Zod schemas, validation, tRPC input/output, z.enum, z.object, z.void
**MUST READ**:
- [rules/libraries/ZOD.md](rules/libraries/ZOD.md) - Zod best practices, enum validation, empty inputs

**Search for**: `Grep pattern="z\.|zod|schema" path="rules/"`

#### When Working on DATABASE (Drizzle ORM)
**Triggers**: Migrations, queries, schema changes, PostgreSQL operations
**MUST READ**:
- [apps/milo/src/server/database/](apps/milo/src/server/database/) - Schema definitions
- [apps/milo/drizzle/](apps/milo/drizzle/) - Migration files

**CRITICAL**: Before writing ANY database operations, check the schema at `apps/milo/src/server/database/schema/` to understand table structures.

**Search for**: `Grep pattern="drizzle|schema|migration|database" path="apps/milo/"`

#### When DEBUGGING or INVESTIGATING ERRORS
**Triggers**: tRPC errors, console errors, unexpected behavior, tracing issues, bug investigation
**MUST READ**:
- [rules/DEBUGGING.md](rules/DEBUGGING.md) - Investigation protocol, error tracing, common patterns

**Search for**: `Grep pattern="debug|trace|investigate|error" path="rules/"`

#### When Working on tRPC ROUTERS or API DESIGN
**Triggers**: Creating routers, platform-specific APIs, multi-platform services
**MUST READ**:
- [rules/API_PATTERNS.md](rules/API_PATTERNS.md) - Platform-specific routers, tRPC structure

**Search for**: `Grep pattern="router|trpc|platform" path="rules/"`

#### When Working on URL STATE or SEARCH PARAMS
**Triggers**: nuqs, search params, filters, sorting, URL-based state
**MUST READ**:
- [rules/URL_STATE.md](rules/URL_STATE.md) - nuqs patterns, sort format, server-side parsing

**Search for**: `Grep pattern="nuqs|searchParams|sort" path="rules/"`

#### When Working on INFRASTRUCTURE or DEPLOYMENT PLATFORMS
**Triggers**: Coolify, Kubernetes, AWS ECS, platform statuses, infrastructure monitoring
**MUST READ**:
- [rules/PLATFORM_NOMENCLATURE.md](rules/PLATFORM_NOMENCLATURE.md) - Official platform terminology, status values

**Search for**: `Grep pattern="coolify|k8s|aws|platform" path="rules/"`

### Quick Reference: All Rule Files

| File | When to Use |
|------|-------------|
| `rules/CODE_STYLE.md` | Formatting, JSX, conditional rendering, composition patterns, React keys |
| `rules/NAMING_CONVENTIONS.md` | Interface naming, booleans, enums (UPPERCASE), dialog/sheet naming, **no Details/Info/Data suffixes** |
| `rules/COMPONENT_PATTERNS.md` | Component organization, utilities, UI standards |
| `rules/TESTING.md` | E2E tests, tRPC testing, Playwright browser tests |
| `rules/DOMAIN_ENTITIES.md` | Entities, validation, mappers, repositories |
| `rules/AUTHENTICATION.md` | Better-Auth, API keys, OAuth, sessions |
| `rules/DEBUGGING.md` | Error investigation, tracing, debugging protocol |
| `rules/PACKAGE_EXPORTS.md` | Module exports and package organization |
| `rules/API_PATTERNS.md` | Platform-specific tRPC routers, API structure |
| `rules/URL_STATE.md` | nuqs, search params, sort format (`?sort=field.desc`) |
| `rules/PLATFORM_NOMENCLATURE.md` | Coolify/K8s/AWS official terminology, status values |
| `rules/libraries/ZOD.md` | Zod best practices, enum validation, empty inputs |
| `rules/libraries/REACT_TABLE.md` | TanStack React Table, table meta types, column definitions |
| `rules/libraries/NEXTJS.md` | "use client"/"use server" directives, env vars (no duplication) |
| `rules/libraries/BUN.md` | Bun runtime standards and patterns |
| `rules/libraries/TAILWINDCSS.md` | Tailwind CSS patterns and utilities |
| `rules/libraries/MOTION.md` | Animation and motion patterns |

### Rule Search Commands

```bash
# Find all rule files
Glob pattern="rules/**/*.md"

# Search for specific topic in rules
Grep pattern="your-topic" path="rules/"

# Find rules mentioning a specific pattern
Grep pattern="Sheet|Dialog" path="rules/" output_mode="content"
```

**Remember**: When you encounter unfamiliar patterns or are unsure about conventions, ALWAYS search the rules directory first before implementing

### Rule Maintenance (MANDATORY)

When adding, modifying, or deleting ANY rule:

1. **Search ALL rule files first** - Use `Grep pattern="topic" path="rules/"` to find all occurrences across all rule files
2. **Apply changes consistently** - If a rule exists in multiple files, update ALL of them
3. **Check for duplicates** - Before adding a new rule, search to ensure it doesn't already exist elsewhere
4. **Update cross-references** - If renaming or moving rules, update all references in other files
5. **Verify completeness** - After changes, run `Grep` again to confirm all instances were updated

**Key principle**: Rules may be split across multiple files. ANY rule change requires searching and updating ALL relevant files to maintain consistency.

## Critical Instructions

### Turborepo Monorepo Workflow (MANDATORY)
- **Always run scripts from workspace root** - Execute all scripts using `bun` from the monorepo root directory
- **Use --filter flag for specific packages** - When targeting specific apps or packages, use `bun turbo <script> --filter=<package-name>`
- **Use turbo for workspace-wide commands** - For workspace-wide commands, use `bun turbo <script>`
- **Never navigate into package directories** - Stay in the root and use filters instead of `cd` commands
- **ALWAYS verify scripts exist in package.json first** - Before running any script, check the relevant package.json files

### Quality Assurance (MANDATORY)
- **ALWAYS run lint and typecheck after EVERY code change** - `bun run lint` and `bun run typecheck`
- **ALWAYS run unit tests after EVERY code change** - `bun run test`
- **ZERO warnings and errors policy** - Fix ALL warnings and errors before considering a task complete
- **Run build verification** - Always run `bun run build` to ensure successful compilation
- **ALWAYS test UI changes with Playwright** - After ANY UI change, verify the changes work correctly

### Development Server Management
- **NEVER start development servers automatically** - Only start servers when explicitly requested by the user

### Version Control
- **NEVER commit changes unless explicitly asked** - Only commit when explicitly asked
- **NEVER use git for reverting changes** - Always revert changes manually by editing files directly

### Docker Operations
- **Docker system cleanup** - Always run `docker system prune -f` before testing with Docker

### Drizzle Database Migrations (MANDATORY)
- **Use Drizzle Kit for migrations** - Use `bun turbo db:generate --filter=@meeboter/milo -- --name <migration_name>` to generate
- **Apply migrations** - Use `bun turbo db:migrate --filter=@meeboter/milo` to apply
- **Database drift requires user consent** - Get explicit user consent before running destructive operations
- **Development vs production** - Database reset ONLY on local development databases

### Database Query Optimization (MANDATORY)
- **ALWAYS select only needed fields** - Use Drizzle's select to fetch only required columns
- **Avoid fetching entire rows** - Select specific columns for better performance
- **Use proper indexes** - Check schema for available indexes before writing queries

## Agent Behavior

- **Always work from monorepo root** - Execute all commands using `bun turbo` with `--filter` flag
- **Verify scripts before execution** - Check package.json files to confirm scripts exist
- **Read README.md files** first for product requirements
- **Research latest documentation** when working with third-party libraries
- **Proactively search the web** for solutions using WebSearch tool
- **NEVER use Bash for file editing or writing** - Always use Claude Code tools (Read, Edit, Write) instead of CLI commands

### Proactive Web Search During Investigation, Debugging, and Testing (MANDATORY)
When investigating bugs, debugging errors, or testing features, ALWAYS search the web proactively:
- **Search error messages** - Find known issues, solutions, and workarounds
- **Search before getting stuck** - If stuck for more than a few minutes, search immediately
- **Search for library-specific patterns** - Find correct usage and common pitfalls
- **Search for version-specific issues** - Include version numbers when relevant
- **Consult official library documentation** - ALWAYS check official docs, GitHub wiki, and GitHub issues/discussions

## Cross-Cutting Patterns (CRITICAL)

### tRPC Server-Side Prefetching (MANDATORY)
Always prefetch tRPC queries in async Next.js server pages:

```typescript
import { prefetch, trpc } from "@/lib/trpc/server";

export default async function Page({ searchParams }) {
    const search = searchParamsCache.parse(await searchParams);

    prefetch(trpc.bots.list.queryOptions({
        status: search.status || [],
        take: search.size,
    }));

    return <ClientComponent />;
}
```

**Benefits**: Eliminates request waterfalls, data available immediately.

### Zod Essential Patterns (MANDATORY)
```typescript
// Empty inputs - use z.void()
export const myInputSchema = z.void();  // NOT z.object({})

// Enums - use z.enum()
const schema = z.enum(["value1", "value2"]);

// Records - specify both key and value
z.record(z.string(), z.string())  // NOT z.record(z.string())
```

### Next.js "use client" Directive (MANDATORY)
- **Only add when needed**: hooks, browser APIs, event handlers
- **Never add to**: column definitions, type files, utilities, constants
- **Push boundaries down**: Keep as much as possible server-side

---

## General Code Standards

- Use TypeScript for all `.ts` and `.tsx` files
- Avoid inline CSS
- Don't push changes until tests pass
- Always write code in English
- **NEVER write obvious comments** - Comments should explain "why", not "what"
- **Use parentheses/commas in comments** - Use `(context)` or `, context` instead of `- context` for clarifying context
- **CRITICAL: Use box-drawing lines for section headings** - Use `// ─── Section Name ────────────────────` format
- **Use react-hook-form + zod + shadcn** for all forms
- **NEVER use `any` type** - Use `Record<string, unknown>`, `unknown`, or specific interfaces
- **NEVER use linter suppression comments** - Fix the underlying issue instead
- **NEVER use nested ternary expressions** - Use if-else or helper functions
- **ALWAYS implement type-safe code** - Prioritize type safety in all implementations
- **Use ternary for conditional JSX rendering** - Use `{condition ? (<Component />) : null}` instead of `{condition && <Component />)}`

### Naming Conventions (CRITICAL) - See `rules/NAMING_CONVENTIONS.md`
- **No redundant suffixes** - NEVER use "Details", "Info", "Data", "Response", "List" suffixes
- **No redundant type suffixes** - Don't repeat units in names when documented (use `timeout` not `timeoutMs` if JSDoc says "in milliseconds")
- **Boolean prefixes required** - ALWAYS use `is`, `has`, `should`, `can`, `will` prefixes for booleans
- **Enums UPPERCASE** - All enum values must be UPPERCASE (`"ACTIVE"` not `"active"`)
- **Request/Response for use cases** - Domain layer interfaces use `Request`/`Response` suffixes
- **Input/Output for tRPC** - tRPC schemas use `Input`/`Output` suffixes
- **Dialog/Sheet naming** - Action-first for CRUD (`CreateBotDialog`), entity-first for view (`BotDialog` not `BotDetailsDialog`)

## File Structure & Organization

- **NEVER create index.ts files for re-exports only** - Import directly from source files
- **One component per file** - Each component should have its own dedicated file
- **Documentation filenames MUST be uppercase** - All `.md` files use UPPERCASE names

## Error Handling

- **Functions should not be entirely wrapped in try-catch** - Let errors propagate to calling code
- **Use try-catch at function usage points** - Wrap individual function calls where used

## Workspace & Monorepo Configuration

### Application Structure
- **Milo app** - Port 3000 (`http://localhost:3000`) - Main API and dashboard
- **Bots app** - Platform-specific meeting bots (Google Meet, Microsoft Teams, Zoom)

### Next.js Navigation
- **NEVER use window.location** - Use Next.js `Link` component or `useRouter` hook
- **Use Link for navigation links** - Use `router.push()` only for programmatic navigation

## Development Testing Instructions

### Manual Testing Workflow
1. **Development Servers** - NEVER start automatically, only when explicitly requested
   - Milo app: `bun turbo dev --filter=@meeboter/milo` (port 3000)

### Development URLs
- **Default**: Use `localhost:3000` for Milo
- **API Docs**: `http://localhost:3000/docs`

## Common Commands Reference

### Quality Assurance Commands
```bash
# MANDATORY after every code change (from root)
bun run lint           # Fix ALL warnings and errors
bun run typecheck      # Fix ALL warnings and errors
bun run test           # Run all unit tests
bun run build          # Verify build success

# Docker cleanup
docker system prune -f    # Clean up before testing
```

### Turborepo Workflow Commands
```bash
# Package-specific operations
bun turbo dev --filter=@meeboter/milo        # Start Milo dev server

# Workspace-wide commands
bun run lint
bun run typecheck
bun run build
```

### Database Commands
```bash
# Generate a new migration
bun turbo db:generate --filter=@meeboter/milo -- --name <migration_name>

# Apply migrations
bun turbo db:migrate --filter=@meeboter/milo

# Open Drizzle Studio (database UI)
bun turbo db:studio --filter=@meeboter/milo
```

## Session Completion (MANDATORY)

### Manual Testing Plan Creation
After completing all implementation tasks in a session, you MUST:

1. **Create a comprehensive testing plan** - Document all changes made and how to manually verify them
2. **Verify each feature works end-to-end** - Test the full user flow, not just individual components
3. **Document test results** - Take screenshots of successful tests when relevant
4. **Report any issues found** - If tests fail, document the issue and fix before considering the session complete

**Key principle**: No session is complete until all changes have been manually tested and verified working.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Payme-Works) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
