## autonomous-agents

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Autonomous Agents is a TypeScript/Next.js application where users create entities (teams or aides) containing AI agents that run continuously to fulfill a mission. Entities have hierarchical agents (leads run continuously, subordinates spawn on-demand) that collaborate, extract knowledge from work sessions, and proactively deliver insights to users.

**Terminology Note**:
- "Entity" is the unified concept for teams and aides. Teams have multiple agents; aides have a single agent.
- "Subordinate" refers to non-lead agents that report to the lead.
- "Worker" refers to the background process (`src/worker/`) that runs agent cycles.

## Commands

```bash
# Start Postgres, run migrations, start web + worker. This is the preferred development workflow
docker compose up
```

Alternatively, if you want to use `npm` directly

```bash
# Development
npm run dev          # Start Next.js dev server
npm run build        # Build for production
npm run lint         # Run ESLint

# Background Worker (for autonomous operation)
npx ts-node --project tsconfig.json src/worker/index.ts  # Start worker process

# Database
docker compose up -d              # Start PostgreSQL (port 5433)
npx drizzle-kit generate          # Generate migrations from schema changes
npx drizzle-kit migrate           # Apply migrations
npx drizzle-kit studio            # Open Drizzle Studio UI
```

## Architecture

### Foreground/Background Architecture

The system separates user interactions (foreground) from agent work (background):

**Conversation Modes**:
- **Foreground** (`mode='foreground'`): User-Agent interaction (permanent, UI-visible, one per agent). Human user sends messages, agent responds.
- **Background** (`mode='background'`): Agent work sessions (persistent, internal only, one per agent). Agent processes tasks, uses tools, extracts knowledge. Automatic compaction via summary messages.

**Memories vs Knowledge Items**:
- **Memories**: User interaction context (preferences, past requests). Extracted from foreground conversations. Sent to LLM in **foreground only**.
- **Knowledge Items**: Professional knowledge base (domain expertise, techniques, patterns, facts). Extracted from background conversations (`sourceConversationId`). Sent to LLM in **background only**.

### Core Components

**Agent Runtime** (`src/lib/agents/`)
- `agent.ts` - Agent class with foreground/background separation:
  - `handleUserMessage()` - Foreground: quick ack + queue task, returns immediately
  - `runWorkSession()` - Background: process queue in background conversation, extract knowledge, decide briefing
  - `processTask()` - Per-task processing with tools in background conversation
  - `extractKnowledgeFromConversation()` - Post-session professional learning (via `knowledge-items.ts`)
  - `decideBriefing()` - Lead briefing decision after work session
- `memory.ts` - Memory extraction from user conversations using `generateObject()`
- `knowledge-items.ts` - Knowledge extraction from background conversations (professional knowledge)
- `compaction.ts` - Conversation compaction via summary messages
- `taskQueue.ts` - Task queue operations (queue, claim, complete)
- `llm.ts` - Provider abstraction (OpenAI, Anthropic, Gemini). Looks up user's encrypted API keys, falls back to env vars

**Database** (`src/lib/db/`)
- PostgreSQL with Drizzle ORM
- Schema: users, entities, agents, conversations (with mode), messages (with toolCalls, previousMessageId), memories, knowledgeItems (with sourceConversationId), agentTasks, inboxItems
- `drizzle.config.ts` points to `src/lib/db/schema.ts`
- `inbox_items` stores `userId` and `agentId` only; entity is derived via the agent relation; types are `briefing` or `feedback`

**Background Worker** (`src/worker/runner.ts`)
- Event-driven + timer-based execution:
  - **Event-driven**: Tasks queued via `notifyTaskQueued()` trigger immediate processing
  - **Timer-based**: Leads scheduled for daily proactive runs via `leadNextRunAt`
- Subordinates are purely reactive (only triggered when work in queue)
- Leads are proactive (daily trigger to further mission)
- Calls `agent.runWorkSession()` for background conversation task processing

**Authentication** (`src/lib/auth/config.ts`)
- NextAuth.js with passwordless magic links
- DrizzleAdapter for session persistence
- In dev, magic links log to console

### Data Flow

**Foreground (User Interaction)**:
1. User sends message to lead
2. Agent loads MEMORIES (user context)
3. Generates quick contextual acknowledgment
4. Queues task for background processing
5. Returns acknowledgment immediately

**Background (Work Session)**:
1. Task picked up from queue (event-driven or scheduled)
2. Agent uses background conversation (persistent, with automatic compaction)
3. Agent loads KNOWLEDGE ITEMS (professional knowledge)
4. Processes task with tools in background conversation
5. After queue empty: lead appends a briefing-decision turn in the background conversation
6. Lead only: may call `createBriefing` (writes `briefings` + inbox item; no foreground message)
7. Leads can call `requestUserInput` to create a `feedback` inbox item and append the full message to the foreground conversation
8. Leads can call `listBriefings` (metadata only) and `getBriefing` (full content) in foreground conversations to answer user questions about briefings
9. After briefing-decision turn: extracts knowledge from conversation
10. Next run scheduled

### Key Patterns

- **Path alias**: `@/*` maps to `./src/*`
- **Mock mode**: Set `MOCK_LLM=true` in `.env.local` to run without real API calls
- **Encrypted API keys**: User API keys stored encrypted in `userApiKeys` table
- **Agent hierarchy**: Leads have `parentAgentId = null`, subordinates reference their lead
- **Memories vs Knowledge Items**: Memories store user interaction context. Knowledge items are the agent's professional knowledge base.
- **Conversation compaction**: Summary messages compress old context via linked list (`previousMessageId`). Context loading returns latest summary + recent messages.
- **Professional growth**: Knowledge items accumulate as expertise from background work sessions
- **Atomic turns + retries**: Persist user+assistant messages together; only complete tasks after turn is saved, with exponential backoff for task retries
- **Worker assumption**: Single worker per agent queue; concurrent workers would require locking changes
- **Backoff gating**: `agents.backoffNextRunAt` + `backoffAttemptCount` gate retries for queued tasks and scheduled runs

## Autonomous Operation

For entities to run autonomously and deliver proactive insights:

1. **Entity status must be 'active'** - Set via UI or database
2. **Worker process must be running** - Start separately from dev server:
   ```bash
   npx ts-node --project tsconfig.json src/worker/index.ts
   ```
3. **Event-driven**: Tasks queued trigger immediate processing via `notifyTaskQueued()`
4. **Timer-based**: Leads auto-scheduled for daily proactive runs
5. Leads can delegate to subordinates and push briefings to user inbox

## Workflow Preferences

**IMPORTANT - Plans Location**: ALL implementation plans go in `docs/plans/` with naming `<next-incrementing-number>-<topic>.md`. Never use `.claude/plans/` or any other location. This is non-negotiable. The incrementing number in front of the plan document is important must be the next line; this allows sorting plans in order they were applied to repositories, effectively being able to track changes made over time.

**Subagent Usage**: Always prefer to use subagents for implementation tasks:
1. **Implementation subagent** - Writes code, creates files, installs dependencies
2. **Review subagent** - Reviews implementation against plan/standards, then commits changes

**Keeping CLAUDE.md Updated**: Whenever working in this repository and something sounds like it's worth keeping in mind for the future (patterns, gotchas, decisions, learnings), update this CLAUDE.md file immediately.

**NEVER EVER rename files in docs/plans**: When you're engaged in refactoring to rename things, never ever rename documents in `docs/plans`. These documents represent a snapshot of the development of this codebase over time, and it doesn't make sense to rename the previous documents as the new plan document will fit nicely in the evolution timeline of this codebase, together with other plans documents.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tnsengimana) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
