## cursor-next-template

> - Always use `bun`; do NOT use `pnpm`,`npm`,`yarn`

# Cursor Next.js Template Project

## IMPORTANT
- Always use `bun`; do NOT use `pnpm`,`npm`,`yarn`

## Naming Conventions
- Always use kebab-case for component file naming

## Auto-linting
Automatic linting runs via Cursor hooks after each file edit. The `.cursor/hooks.json` configuration triggers `bunx biome check --write` on edited files automatically (equivalent to `bun run lint:fix` but runs on individual files), so manual linting is not required.

## Debuggable
All components and code must be easy to debug on development environment
- Type safety
- Descriptive variable/function/component names
- Avoid abbreviations unless widely understood
- Strategic console.log/console.error for key flows
- Single responsibility per function/component
- Clear separation of concerns

### Client-Side Debugging

#### Source Maps
- Source maps are enabled in development mode (`tsconfig.json` and `next.config.ts`)
- Original TypeScript code is visible in browser DevTools
- Use `.vscode/launch.json` for Cursor IDE debugging with breakpoints

#### Debug Utilities (`lib/debug.ts`)
Use debug utilities instead of raw `console.log` for consistent, development-only logging:

```typescript
import { debug, debugError, debugTime, debugGroup } from "@/lib/debug";

// Basic debug logging
debug({ component: "LoginForm", function: "handleSubmit" }, "User submitted form", { email });

// Error logging with stack traces
debugError({ component: "AuthProvider" }, error, { userId });

// Performance timing
debugTime("fetchUserData", { component: "UserProfile" });
// ... async operation ...
debugTimeEnd("fetchUserData", { component: "UserProfile" });

// Grouped logs
debugGroup("Authentication Flow", () => {
  debug("Step 1: Validate credentials");
  debug("Step 2: Create session");
});
```

#### Development Helpers (`lib/dev-utils.ts`)
Use development-only utilities to ensure code is stripped in production:

```typescript
import { isDev, devOnly, devAssert, devWarn } from "@/lib/dev-utils";

// Check development mode
if (isDev()) {
  // Development-only code
}

// Execute function only in development
const result = devOnly(() => expensiveDebugOperation(), undefined);

// Assertions in development
devAssert(user !== null, "User must be authenticated");

// Development warnings
devWarn("Deprecated API usage", { component: "LegacyComponent" });
```

#### Error Boundaries (`app/components/debug/error-boundary.tsx`)
Wrap components with ErrorBoundary to catch and display errors gracefully:

```typescript
import { ErrorBoundary } from "@/app/components/debug/error-boundary";

<ErrorBoundary
  onError={(error, errorInfo) => {
    // Custom error handling
  }}
>
  <YourComponent />
</ErrorBoundary>
```

#### Cursor IDE Debugging
1. **Server-side debugging**: Use "Next.js: debug server-side" configuration
2. **Client-side debugging**: Use "Next.js: debug client-side" configuration (requires Chrome/Edge)
3. **Full stack debugging**: Use "Next.js: debug full stack" for both server and client
4. Set breakpoints directly in TypeScript files - they will work with source maps

#### Best Practices
- Use `debug()` instead of `console.log()` for development logging
- Use `debugError()` for error logging with context
- Wrap major components with ErrorBoundary
- Use `debugTime()`/`debugTimeEnd()` for performance debugging
- All debug utilities automatically disable in production builds
- Use `isDev()` checks for development-only features

## DEVELOPMENT
- Use `list_mcp_resources` tool to list down available tools first
- Check `.cursor/tmp/hooks.log` for hooks log information everytime modifying the codebase
- Check `.cursor/tmp/runner.log` for runner `bun dev` log information related to server-side
- Update USER-CHOICE.md if user setup new configurations (create one if NOT EXIST; refer to USER-CHOICE-TEMPLATE.md)
- for AI development, view the logs on .devtools/generations.json

## Beads Issue Tracking Integration

Beads (`bd`) is a git-backed issue tracker designed for AI-assisted development workflows. Issues are stored in `.beads/issues.jsonl` and synced via git.

### Session Start Workflow (MANDATORY)

**Always start by syncing and reading issues:**

1. **Sync issues from remote:**
   ```bash
   bd sync --pull
   ```

2. **Read `.beads/issues.jsonl` directly** to understand:
   - Current open issues and their status
   - Dependencies between issues
   - Priorities and labels
   - Issue descriptions

3. **Identify work to do:**
   - Find issues with status `ready` or `open`
   - Check dependencies - only work on issues whose dependencies are `closed`
   - Prioritize by `priority` field (`critical` > `high` > `normal` > `low`)

4. **Start working on an issue:**
   ```bash
   bd update <issue-id> --status in_progress
   ```

**Helper commands:**
```bash
# Quick status check
.cursor/hooks/beads-status.sh

# Interactive work starter
.cursor/hooks/beads-start-work.sh

# Get context for AI
.cursor/hooks/beads-context.sh
```

### During Work Patterns

**When working on an issue:**

- **Read issue details:** `bd show <issue-id>`
- **Check dependencies:** Ensure all dependencies are `closed` before starting
- **Update progress:** Add notes with `bd update <issue-id> --notes "progress update"`

**When creating sub-tasks:**

- **Create new issue:** `bd create "Sub-task: Description" "Details"`
- **Set dependencies:** `bd depends <parent-issue-id> <sub-task-id>`
- **Add labels:** `bd label <issue-id> frontend|backend|bug|feature`
- **Set priority:** `bd priority <issue-id> low|normal|high|critical`

**When breaking down large tasks:**

1. Create parent issue with overall goal
2. Create sub-issues for specific implementation details
3. Set dependencies: `bd depends <parent-id> <sub-id>`
4. Work on sub-issues first, then parent

### Issue Creation Best Practices

**Create granular, actionable issues:**

```bash
bd create "Type: Brief description" "**Problem:**
[Describe the problem]

**Location:**
- File: \`path/to/file.ts\`
- Lines: X-Y

**Root Cause:**
[Why does this exist?]

**Proposed Solution:**
[How to fix/implement]

**Files to modify:**
- \`file1.ts\`
- \`file2.tsx\`"
```

**Add metadata:**

```bash
# Add labels
bd label <issue-id> bug|feature|frontend|backend|api|ui

# Set priority
bd priority <issue-id> low|normal|high|critical

# Set dependencies
bd depends <issue-id> <dependency-id>
```

**Label conventions:**
- `bug` - Bug fixes
- `feature` - New features
- `frontend` - Frontend work
- `backend` - Backend work
- `api` - API changes
- `ui` - UI/UX changes
- `refactor` - Code refactoring
- `docs` - Documentation

**Priority levels:**
- `low` - Nice to have, not urgent
- `normal` - Standard priority (default)
- `high` - Important, should be done soon
- `critical` - Urgent, blocking issue

### Session End Workflow (MANDATORY)

**Before ending work:**

1. **Update issue status:**
   ```bash
   # For completed work
   bd update <issue-id> --status done
   bd close <issue-id>
   
   # For work in progress
   bd update <issue-id> --notes "Progress: what was done, what remains"
   ```

2. **Create issues for remaining work:**
   ```bash
   bd create "Task: Description" "Details"
   bd label <issue-id> <labels>
   bd priority <issue-id> <priority>
   ```

3. **Sync issues:**
   ```bash
   bd sync
   ```

4. **Git workflow:**
   ```bash
   git pull --rebase
   bd sync  # Ensure beads are synced
   git add .beads/issues.jsonl  # Explicitly stage beads changes
   git commit -m "feat: [issue-id] description"
   git push
   ```

### Beads Commands Reference

**Essential commands:**

```bash
# Create issue
bd create "Title" "Description"

# List issues
bd list --open
bd list --status in_progress
bd list --status ready

# Show issue details
bd show <issue-id>

# Update status
bd update <issue-id> --status in_progress|done|ready
bd close <issue-id>

# Add metadata
bd label <issue-id> <label1> <label2>
bd priority <issue-id> <priority>
bd depends <issue-id> <dependency-id>

# Sync
bd sync
bd sync --pull
bd sync --push
```

**See also:**
- `.cursor/commands/start-work.md` - Session start workflow
- `.cursor/commands/end-work.md` - Session end workflow
- `.cursor/commands/beads-create-issue.md` - Issue creation guide
- `.cursor/commands/use-beads.md` - Complete beads reference

### Key Integration Points

1. **Manual Sync Required** - Cursor hooks don't support session events, so agent must run `bd sync` manually at session start/end
2. **Read JSONL Directly** - Agent should parse `.beads/issues.jsonl` to understand current state
3. **Respect Dependencies** - Only work on issues whose dependencies are `closed`
4. **Update Status During Work** - Use `bd update --status in_progress` when starting
5. **Close When Done** - Always `bd close <issue-id>` after completing work
6. **Sync Before Git Push** - Always `bd sync` before `git push`

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **Update beads issues:**
   ```bash
   # Close completed issues
   bd close <issue-id>
   
   # Update in-progress issues
   bd update <issue-id> --notes "Progress update"
   
   # Create issues for remaining work
   bd create "Task: Description" "Details"
   ```

2. **Sync beads issues:**
   ```bash
   bd sync
   ```

3. **File issues for remaining work** - Create issues for anything that needs follow-up (if not done in step 1)
4. **Run quality gates** (if code changed) - Tests, linters, builds
5. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd sync  # Ensure beads are synced
   git add .beads/issues.jsonl  # Explicitly stage beads changes
   git commit -m "feat: [issue-id] description"
   git push
   git status  # MUST show "up to date with origin"
   ```
6. **Clean up** - Clear stashes, prune remote branches
7. **Verify** - All changes committed AND pushed:
   ```bash
   git status
   bd list --open  # Verify issue status
   ```
8. **Hand off** - Provide context for next session, including open issues

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AureliusIvan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AureliusIvan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
