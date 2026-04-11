## template-11-2025

> > **Universal machine instructions for all AI coding agents**

# AI Agent Development Guidelines

> **Universal machine instructions for all AI coding agents**  
> Standard format used by 20,000+ repositories for AI-assisted development

---

## ⚠️ CRITICAL RULES - READ FIRST

### Documentation Files
**ABSOLUTELY FORBIDDEN** to create documentation files (README.md, CHANGELOG.md, CONTRIBUTING.md, etc.) unless **EXPLICITLY** requested by the user. This includes:
- ❌ Summary files of changes made
- ❌ Documentation of implemented features
- ❌ Markdown files explaining your work
- ❌ Any kind of "here's what I did" documents

**ONLY create documentation files when the user specifically says:**
- "Create a README"
- "Document this feature"
- "Write documentation for..."
- Or similar explicit requests

When you finish work, simply report what you did in the chat. NO FILES.

---

## Purpose

This file provides comprehensive guidelines for AI agents working on this project. It covers:
- **Multi-agent coordination** using Convex real-time backend
- **Spec-kit workflow** from specification to implementation
- **Convex-specific patterns** for queries, mutations, and actions
- **Contract-first development** with type safety
- **MCP integration** for external tool access

## Project Stack

**Frontend**: Next.js 15.5.6 + React 19.2.0 + Tailwind CSS 4 + Radix UI  
**Backend**: Convex 1.28.2 with Convex Auth  
**Type Safety**: TypeScript 5.9.3 with strict mode  
**Testing**: Jest + Playwright  
**Package Manager**: pnpm

---

## Spec-Kit Workflow

### Commands Available

1. **`/speckit.specify`** - Create feature specification from natural language
2. **`/speckit.plan`** - Generate implementation plan with tech stack
3. **`/speckit.tasks`** - Break down into dependency-ordered tasks
4. **`/speckit.implement`** - Execute tasks with TDD approach
5. **`/speckit.orchestrate`** - Multi-agent coordinated implementation
6. **`/speckit.clarify`** - Resolve ambiguities in specifications
7. **`/speckit.analyze`** - Check consistency across spec/plan/tasks
8. **`/speckit.checklist`** - Generate quality validation checklists
9. **`/speckit.constitution`** - Define/update project principles

### Workflow Phases

#### Phase -1: Contract Definition (REQUIRED)
**Before any implementation:**
1. Define function contracts in `specs/###-feature/contracts/convex/`
2. Use Convex validators (`v.*`) for type safety
3. Write failing contract tests
4. Generate TypeScript types
5. Frontend and backend can now work in parallel

**Example Contract:**
```typescript
// specs/001-chat/contracts/convex/messages.ts
import { v } from "convex/values";

export interface MessagesAPI {
  list: {
    args: { channelId: typeof v.id("channels") };
    returns: typeof v.array(v.object({
      _id: typeof v.id("messages"),
      content: typeof v.string(),
      authorId: typeof v.id("users")
    }));
    description: "List messages in a channel";
  };
}

export const messagesValidators = {
  list: {
    args: { channelId: v.id("channels") },
    returns: v.array(v.object({
      _id: v.id("messages"),
      content: v.string(),
      authorId: v.id("users")
    }))
  }
};
```

#### Phase 0: Schema Definition
1. Define database schema in `convex/schema.ts`
2. Use `defineTable` with validators
3. Define indexes for query optimization
4. Validate schema compiles

#### Phase 1: Parallel Implementation
**Backend Agent:**
- Implement Convex functions using contract validators
- Follow `convex/rules` in `.cursor/rules/convex_rules.mdc`
- Make contract tests pass

**Frontend Agent:**
- Implement React components with Convex hooks
- Use generated types from contracts
- Follow Next.js 15 patterns

#### Phase 2: Testing
- Contract tests (already written!)
- Integration tests for Convex functions
- Component tests for React components
- E2E tests for critical user flows

---

## Convex Development Patterns

### Function Syntax (NEW - Always Use This)

```typescript
import { query, mutation, action } from "./_generated/server";
import { v } from "convex/values";

// Query
export const list = query({
  args: { channelId: v.id("channels") },
  returns: v.array(v.object({
    _id: v.id("messages"),
    content: v.string()
  })),
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .collect();
  }
});

// Mutation
export const send = mutation({
  args: {
    channelId: v.id("channels"),
    content: v.string()
  },
  returns: v.id("messages"),
  handler: async (ctx, args) => {
    return await ctx.db.insert("messages", {
      channelId: args.channelId,
      content: args.content
    });
  }
});

// Action (for external APIs)
export const sendEmail = action({
  args: { to: v.string(), subject: v.string() },
  returns: v.object({ success: v.boolean() }),
  handler: async (ctx, args) => {
    // Can call external APIs
    const response = await fetch("https://api.sendgrid.com/...");
    return { success: response.ok };
  }
});
```

### Type Safety Rules

1. **Always include args and returns validators**
2. **Use `v.id("tableName")` for document IDs**
3. **Use `v.null()` for void returns**
4. **Import validators from contracts** when they exist
5. **Use TypeScript strict mode** (`Id<"tableName">`)

### Query Optimization

```typescript
// ❌ WRONG: No index, slow table scan
const messages = await ctx.db
  .query("messages")
  .filter((q) => q.eq(q.field("channelId"), channelId))
  .collect();

// ✅ CORRECT: Use index for fast lookup
const messages = await ctx.db
  .query("messages")
  .withIndex("by_channel", (q) => q.eq("channelId", channelId))
  .collect();
```

### Internal vs Public Functions

```typescript
// Public API (exposed to clients)
export const publicQuery = query({ ... });

// Internal API (server-side only)
import { internalQuery } from "./_generated/server";
export const internalQuery = internalQuery({ ... });

// Call internal functions
import { internal } from "./_generated/api";
await ctx.runQuery(internal.module.internalQuery, { ... });
```

---

## Multi-Agent Coordination

### Agent Coordination System

When multiple agents work on a feature, use **Convex real-time coordination**:

```typescript
// Report agent status
await ctx.runMutation(api.agentCoordination.updateStatus, {
  agentId: "convex-backend-agent",
  featureId: "001-chat-system",
  status: "working",
  currentPhase: "implementation",
  currentTask: "Implementing message queries",
  progress: 40, // 0-100
  message: "Creating query functions"
});

// Subscribe to all agents (primary orchestrator)
const agents = useQuery(api.agentCoordination.subscribeToAgents, {
  featureId: "001-chat-system"
});
// Returns real-time updates as agents work

// Claim a task (prevents duplicate work)
const claim = await ctx.runMutation(api.agentCoordination.claimTask, {
  featureId: "001-chat-system",
  phase: "implementation",
  taskId: "implement-messages",
  agentId: "backend-agent"
});

if (claim.success) {
  // Task is yours, do the work
  await implementMessages();
  
  // Release when done
  await ctx.runMutation(api.agentCoordination.releaseTask, {
    taskClaimId: claim.taskClaimId,
    status: "completed"
  });
}

// Report conflicts
await ctx.runMutation(api.agentCoordination.reportConflict, {
  featureId: "001-chat-system",
  conflictType: "file_modification",
  involvedAgents: ["backend-agent", "test-agent"],
  description: "Both modified convex/messages.ts",
  filePath: "convex/messages.ts"
});
```

### Agent Roles

**Primary Orchestrator:**
- Reads agent manifest (`.specify/templates/agent-manifest.yaml`)
- Spawns subagents for phases
- Monitors progress via Convex subscriptions
- Resolves conflicts
- Reports feature completion

**Convex Backend Agent:**
- Implements queries, mutations, actions
- Follows contract signatures
- Updates agent_status table
- Claims backend tasks

**Next.js Frontend Agent:**
- Implements React components
- Uses Convex hooks (`useQuery`, `useMutation`)
- Claims frontend tasks
- Ensures accessibility

**Test Agent:**
- Generates contract tests
- Generates integration tests
- Generates E2E tests
- Runs test suites
- Reports pass/fail status

### Parallel Work Example

```typescript
// Backend and frontend work simultaneously after contracts defined

// BACKEND: convex/messages.ts
export const list = query({
  args: messagesValidators.list.args,
  returns: messagesValidators.list.returns,
  handler: async (ctx, args) => {
    // Backend reports 40% progress
    await ctx.runMutation(api.agentCoordination.updateStatus, {
      agentId: "backend-agent",
      featureId: "001-chat",
      progress: 40
    });
    
    return await ctx.db.query("messages")...;
  }
});

// FRONTEND: app/chat/MessageList.tsx (works in parallel!)
export function MessageList({ channelId }: { channelId: Id<"channels"> }) {
  // Type-safe from contract!
  const messages = useQuery(api.messages.list, { channelId });
  
  return (
    <div>
      {messages?.map(msg => (
        <div key={msg._id}>{msg.content}</div>
      ))}
    </div>
  );
}
```

---

## MCP Integration

### Available MCP Servers

**Convex MCP Server** (`.specify/mcp/convex-server.ts`):
- `convex_query` - Execute Convex queries
- `convex_mutation` - Execute Convex mutations
- `convex_action` - Execute Convex actions
- `convex_get_schema` - Retrieve database schema
- `convex_list_functions` - List available functions

### Configuration

**For Claude Desktop:**
```json
{
  "mcpServers": {
    "convex": {
      "command": "node",
      "args": ["/workspaces/codespaces-blank/.specify/mcp/dist/convex-server.js"],
      "env": {
        "CONVEX_URL": "https://your-deployment.convex.cloud"
      }
    }
  }
}
```

**For Cursor IDE:**
```json
{
  "servers": {
    "convex": {
      "command": "node",
      "args": ["/workspaces/codespaces-blank/.specify/mcp/dist/convex-server.js"],
      "env": {
        "CONVEX_URL": "https://your-deployment.convex.cloud"
      }
    }
  }
}
```

### Usage in Agent Workflows

```typescript
// Agent queries Convex during development
const messages = await mcp.call_tool("convex_query", {
  functionRef: "api.messages.list",
  args: { channelId: "test-channel" }
});

// Agent mutates data for testing
await mcp.call_tool("convex_mutation", {
  functionRef: "api.messages.send",
  args: {
    channelId: "test-channel",
    content: "Test from agent"
  }
});
```

---

## Best Practices

### 1. Contract-First Always
✅ Define contracts before implementation  
✅ Write failing tests first (TDD)  
✅ Use contract validators in implementation  
❌ Never skip contract definition

### 2. Report Status Frequently
✅ Update status at milestones  
✅ Report when blocked  
✅ Report completion  
❌ Work silently without updates

### 3. Claim Tasks Before Starting
✅ Check if task is available  
✅ Claim before beginning work  
✅ Release when complete  
❌ Start work without claiming

### 4. Handle Conflicts Immediately
✅ Report conflicts when detected  
✅ Wait for resolution before continuing  
✅ Follow constitution for decisions  
❌ Overwrite other agents' work

### 5. Follow Convex Patterns
✅ Use new function syntax  
✅ Include args/returns validators  
✅ Use indexes for queries  
✅ Use internal functions for server-only code  
❌ Use old function syntax  
❌ Skip validators

---

## File Locations

### Agent-Specific Rules
- **Cursor**: `.cursor/rules/convex_rules.mdc` (detailed Convex patterns)
- **Claude**: `CLAUDE.md` (auto-generated from this file)
- **Gemini**: `GEMINI.md` (auto-generated from this file)
- **GitHub Copilot**: `.github/copilot-instructions.md` (auto-generated)
- **Qwen**: `QWEN.md` (auto-generated from this file)
- **Windsurf**: `.windsurf/rules/specify-rules.md` (auto-generated)
- **Kilo Code**: `.kilocode/rules/specify-rules.md` (auto-generated)

### Spec-Kit Artifacts
- **Specs**: `specs/###-feature-name/spec.md`
- **Plans**: `specs/###-feature-name/plan.md`
- **Tasks**: `specs/###-feature-name/tasks.md`
- **Contracts**: `specs/###-feature-name/contracts/convex/`
- **Agent Manifest**: `specs/###-feature-name/agent-manifest.yaml` (generated by `/speckit.orchestrate`)

### Constitution
- **Project Principles**: `.specify/memory/constitution.md`

---

## Quick Reference

### Common Commands
```bash
# Start feature
/speckit.specify "Feature description"

# Create plan
/speckit.plan "Tech stack details"

# Multi-agent implementation
/speckit.orchestrate

# Run tests
npm test

# Start dev
npm run dev
```

### Common Imports
```typescript
// Convex
import { query, mutation, action } from "./_generated/server";
import { v } from "convex/values";
import { api, internal } from "./_generated/api";
import { Id, Doc } from "./_generated/dataModel";

// React + Convex
import { useQuery, useMutation, useAction } from "convex/react";
```

---

## Support

- **Spec-Kit Docs**: `.specify/templates/` 
- **Convex Docs**: https://docs.convex.dev/
- **MCP Setup**: `.specify/mcp/README.md`
- **Multi-Agent Patterns**: See "Multi-Agent Coordination" section above

---

**Last Updated**: Auto-generated on feature changes  
**Format Version**: 1.0.0  
**Standard**: AGENTS.md (20,000+ repositories)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/parseen254)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/parseen254)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
