## opensync-ecosystem

> OpenSync ecosystem development guidelines for building and maintaining plugins


# OpenSync Ecosystem Development Rules

## Overview

OpenSync is a platform for syncing AI coding sessions. This rule covers the ecosystem of plugins and the main application.

## Architecture

### Main Application (opensync/)

- React + Vite frontend
- Convex backend
- WorkOS AuthKit authentication
- Dashboard for viewing sessions

### Plugins (separate repos)

Each plugin syncs sessions from a specific CLI tool:

- `codex-sync` - Codex CLI (source: "codex-cli")
- `claude-code-sync` - Claude Code (source: "claude-code")
- `opencode-sync` - OpenCode (source: "opencode")
- `cursor-cli-sync` - Cursor (source: "cursor")

## Plugin Development

### Source Identifiers

Every plugin MUST have a unique source identifier:

- Used in session `source` field
- Appears in dashboard source filter
- Format: lowercase with hyphens (e.g., "amp-cli", "agent-trace")

### Required Fields for Sessions

```typescript
{
  externalId: string,      // Unique session ID from your plugin
  source: string,          // Your plugin identifier
  promptTokens: number,    // Input tokens
  completionTokens: number,// Output tokens
  cost: number,            // Calculated USD cost
}
```

### Required Fields for Messages

```typescript
{
  sessionExternalId: string, // Links to session externalId
  externalId: string,        // Unique message ID
  role: "user" | "assistant" | "system" | "tool" | "unknown",
}
```

## API Integration

### Endpoints

- `POST /sync/session` - Sync a session
- `POST /sync/message` - Sync a message
- `POST /sync/batch` - Batch sync (preferred for multiple items)
- `GET /health` - Health check

### URL Normalization

Convex URLs must be converted for HTTP endpoints:

```typescript
const httpUrl = convexUrl.replace(".convex.cloud", ".convex.site");
```

### Authentication

Plugins use API key authentication:

```typescript
headers: {
  'Authorization': `Bearer ${apiKey}`,  // Format: osk_xxxxx
  'Content-Type': 'application/json',
  'User-Agent': `plugin-name/${version}`,
}
```

## Convex Backend Patterns

### Idempotent Upserts

Always use upsert pattern for sync operations:

```typescript
const existing = await ctx.db
  .query("sessions")
  .withIndex("by_user_external", (q) =>
    q.eq("userId", userId).eq("externalId", externalId),
  )
  .first();

if (existing) {
  await ctx.db.patch(existing._id, updates);
} else {
  await ctx.db.insert("sessions", data);
}
```

### Indexed Queries

Always use indexes, never filters:

```typescript
// Good
.withIndex("by_user", (q) => q.eq("userId", userId))

// Bad - causes performance issues
.filter((q) => q.eq(q.field("userId"), userId))
```

### Batch Operations

Use batch mutations to reduce write conflicts:

```typescript
export const batchUpsert = internalMutation({
  args: { userId: v.id("users"), sessions: v.array(...) },
  handler: async (ctx, args) => {
    // Process all items in one transaction
  },
});
```

## CLI Commands

Every plugin should implement:

- `login` - Configure API key and Convex URL
- `status` - Show configuration and connection status
- `sync` - Sync sessions to OpenSync
- `logout` - Clear configuration

## Configuration Storage

Standard config location: `~/.plugin-name/config.json`

```json
{
  "convexUrl": "https://app.convex.cloud",
  "apiKey": "osk_xxxxx"
}
```

## Cost Calculation

Use standard pricing lookup by model:

```typescript
const MODEL_PRICING = {
  "gpt-4o": { input: 2.5, cached: 1.25, output: 10.0 },
  "claude-3-5-sonnet": { input: 3.0, cached: 1.5, output: 15.0 },
  // ... per 1M tokens
};
```

## Testing

Before publishing a plugin:

1. Test `login` stores credentials
2. Test `status` shows connection state
3. Test `sync` creates sessions in dashboard
4. Verify source filter includes your plugin
5. Check token counts and costs are accurate

## Git Workflow

Each plugin is a separate repository:

- Commit to the correct repo (check `pwd` first)
- Each repo has its own version and changelog
- Publish independently with `npm publish`

## Documentation

Keep updated in each plugin:

- `README.md` - Installation and usage
- `changelog.md` - Version history
- `files.md` - Codebase structure

## Common Mistakes to Avoid

1. Using `.convex.cloud` instead of `.convex.site` for HTTP endpoints
2. Missing `source` field in session data
3. Using `filter()` instead of `withIndex()`
4. Committing to wrong repository in multi-repo workspace
5. Forgetting to update version before `npm publish`

---
> Source: [waynesutton/opensync](https://github.com/waynesutton/opensync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
