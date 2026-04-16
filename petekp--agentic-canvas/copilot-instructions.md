## agentic-canvas

> A conversational AI workspace that generates UI components on a grid-based canvas.

# Agentic Canvas

A conversational AI workspace that generates UI components on a grid-based canvas.

## Project Status

**Phase:** v0.1 implemented - Canvas + Chat + Spaces + Templates + multi-source integrations

## Quick Start

```bash
pnpm install
cp .env.example .env.local  # Add your OPENAI_API_KEY (+ SUPERMEMORY_API_KEY)
pnpm dev
```

Open http://localhost:3000 and try: "Add a stat tile showing open PRs"

## Tech Stack

- **Next.js 15** + React 19 + TypeScript
- **Zustand** + Immer for state management
- **Vercel AI SDK v6** (`ai`, `@ai-sdk/openai`, `@ai-sdk/react`)
- **Zod v4** for schema validation
- **react-grid-layout** for drag & drop canvas
- **Tailwind CSS v4** for styling

## Quick Start for Agents

When working on this project, read specs in this order:

1. **Types first:** `.claude/plans/primitives-spec-v0.1.md`
   - All TypeScript interfaces live here
   - Commands, events, and protocols defined

2. **Components second:** `.claude/plans/component-schemas-v0.1.md`
   - Config and data shapes for each component
   - Actions the assistant can take

3. **Store third:** `.claude/plans/store-architecture-v0.1.md`
   - Zustand slices and how they compose
   - Command → action → undo/redo flow

4. **Spaces navigation:** `.claude/plans/spaces-navigation-v0.2.md`
   - Spaces grid + space routing
   - Space lifecycle (pin/unpin/cleanup)

5. **Templates:** `.claude/plans/template-primitives-v0.1.md`
   - State-aware template generation

6. **Undo/Redo:** `.claude/plans/undo-redo-system-v2.md`
   - Snapshot-based undo with policies + audit log

7. **assistant-ui tools:** `.claude/plans/assistant-ui-native-tools.md`
   - `makeAssistantTool` patterns + UI rendering

## Key Architecture Decisions

### Why Zustand over Redux?
Simpler API, built-in immer support, easier slice composition.

### Why commands instead of direct mutations?
Every change goes through `CanvasCommand` so we can:
- Record undo/redo automatically
- Let AI and users share the same mutation path
- Validate before applying

### Why API routes instead of direct client calls?
We route data access through `/api/github`, `/api/posthog`, `/api/slack`, `/api/vercel`, and `/api/insights`
to centralize auth, cache/TTL behavior, and response shaping. Memory feedback goes through `/api/memory/feedback`,
and integration availability through `/api/integrations`.

### Why grid-based instead of infinite canvas?
Cognitive load. Fixed grids constrain layout decisions, making both user placement and AI suggestions simpler.

## Patterns to Follow

- **All state through store** — No component-local state for canvas data
- **Commands are immutable** — They describe intent, not mutation
- **Selectors for reads** — Use `shallow` equality to prevent re-renders
- **Computed at render** — Fields like `age` and `isStale` are derived, not stored
- **LLM tools use snake_case** — `add_component`, `move_component` (not camelCase)
- **Internal types use PascalCase** — `CanvasCommand`, `DataLoadingState`
- **Spaces are first-class** — use space APIs and routing (`/spaces`, `/spaces/[id]`)
- **Tools live in canvas-tools** — use `makeAssistantTool` in `src/lib/canvas-tools.tsx`
- **assistant-ui transport is canonical** — use `AssistantChatTransport` + AI SDK `validateUIMessages(...)` at `/api/chat`; avoid hand-rolled message normalization for the standard path
- **Don’t duplicate adapter helpers** — keep tool schema projection via `frontendTools(...)`; avoid adding thin wrappers that mirror SDK/util behavior
- **Runtime seam stays thin** — delegate orchestration/extensions to external pi engine when configured; keep route protocol unchanged

## Telemetry & Headless Debugging

Telemetry is available so agents can debug without a browser.

- Log file (JSONL): `.claude/telemetry/agentic-canvas.log`
- Tail live: `tail -f .claude/telemetry/agentic-canvas.log`
- Filter by source: `rg "api\\.chat|tool\\.set_preference_rules" .claude/telemetry/agentic-canvas.log`
- Query in-memory buffer: `curl "http://localhost:3000/api/telemetry?limit=200"`
- Override path via `.env.local`: `TELEMETRY_LOG_PATH=/custom/path/agentic-canvas.log`
- Standard filter helper: `./scripts/query-telemetry.sh --level error --limit 50`

Notes:
- Secrets are redacted before write.
- Events include API requests/errors and tool start/result/error payloads.

### Event Schema (JSONL)
```json
{
  "ts": "2026-02-09T16:05:16.515Z",
  "level": "info",
  "source": "store.data",
  "event": "fetch_success",
  "data": {
    "componentId": "cmp_abc123",
    "cacheKey": "github:pull_requests:{\"repo\":\"owner/repo\"}",
    "durationMs": 284,
    "ttl": 60000,
    "itemCount": 5
  }
}
```

### Common Queries
```bash
# Recent errors
tail -n 200 .claude/telemetry/agentic-canvas.log | rg "\"level\":\"error\"" | tail -n 50

# Standardized helper
./scripts/query-telemetry.sh --level error --limit 50

# Tool calls + results for a specific tool
rg "\"source\":\"tool\\.add_component\"" .claude/telemetry/agentic-canvas.log | tail -n 20

# Data fetch lifecycle for a component
rg "cmp_abc123" .claude/telemetry/agentic-canvas.log | rg "store\\.data"

# LLM classifier scoring events
rg "\"source\":\"store\\.rules\"" .claude/telemetry/agentic-canvas.log | rg "llm_score"
```

### Event Catalog (prefixes)
- `api.*` — API route requests/errors (chat, github, slack, vercel, rules, briefing)
- `tool.*` — assistant tool execution (start/result/error)
- `store.data.*` — cache hits, fetch start/success/error, invalidations
- `store.rules.*` — rule set/apply + LLM score requests
- `store.canvas.*` — component add/update/remove/move/resize/clear
- `store.undo.*` — undo/redo + batch lifecycle
- `store.audit.*` — audit log append/load/persist errors

## Assistant Tool Commands

**Canvas edits**
- `add_component`, `remove_component`, `move_component`, `resize_component`, `update_component`, `clear_canvas`

**Spaces**
- `create_space`, `switch_space`, `pin_space`, `unpin_space`

**Templates / filters / transforms**
- `generate_template`, `add_filtered_component`, `create_transform`

**Slack helpers**
- `lookup_slack_user`

## AI SDK v6 Notes

Critical patterns for working with Vercel AI SDK v6:

### Tool Definitions
```typescript
const tools = {
  my_tool: {
    description: "...",
    inputSchema: z.object({ ... }),  // NOT "parameters"
    execute: async (params) => { ... }
  }
};
```

### Multi-step Tool Calls
```typescript
import { stepCountIs } from "ai";

streamText({
  model: openai("gpt-4o"),
  tools,
  stopWhen: stepCountIs(5),  // NOT "maxSteps: 5"
});
```

### Tool Part States
Tool invocations in UIMessage.parts use these states:
- `input-streaming` - tool input is streaming
- `input-available` - tool input complete
- `output-available` - tool result ready (check for this, NOT "result")
- `output-error` - tool errored

### Client-side useChat
```typescript
import { useChat } from "@ai-sdk/react";
import { DefaultChatTransport } from "ai";

const { messages, sendMessage, status, stop } = useChat({
  transport: new DefaultChatTransport({ api: "/api/chat" }),
});
// status: "submitted" | "streaming" | "error" | "ready"
```

### Zod v4 Syntax
```typescript
// Use this:
z.record(z.string(), z.unknown())
// NOT: z.record(z.unknown())
```

### assistant-ui + AI SDK boundary
- At `/api/chat`, validate incoming UI messages with `validateUIMessages(...)` before `convertToModelMessages(...)`.
- For historical tool-result ingestion from model messages, unwrap wrapped payloads (`output.value` / `result.value`) before ledger persistence.
- External pi runtime delegation is configured via:
  - `PI_RUNTIME_ENGINE_MODULE`
  - `PI_RUNTIME_ENGINE_EXPORT` (optional)
- Prefer existing utilities over custom protocol code in external engines:
  - pi runtime/model orchestration from `@mariozechner/pi-ai`
  - assistant stream transport from AI SDK `createUIMessageStream(...)` + `createUIMessageStreamResponse(...)`
- Retention runs on traffic via `streamWithPiRuntime(...)` and can also be triggered explicitly at `POST /api/pi/retention`.
- If `PI_RETENTION_API_TOKEN` is set, `POST /api/pi/retention` requires `Authorization: Bearer <token>`.
- Runtime engine diagnostics are available at `GET /api/pi/runtime` when `PI_RUNTIME_DIAGNOSTICS_ENABLED=1`.

## Patterns to Avoid

- Don't use `ComponentState` — it's now `DataLoadingState`
- Don't use `DataTransform` — deferred to v0.2
- Don't create complex triggers — only `session_start` and `time_based` in v0.1
- Don't implement `ErrorRecovery` — deferred to v0.2
- Don't create a `list_components` tool — inject IDs via `CanvasContext`
- Don't use any `view` or `View` naming — fully migrated to `space`/`Space`
- Don't add custom tool executors — use assistant-ui `makeAssistantTool`

## File Structure

```
src/
├── app/
│   ├── api/chat/route.ts           # Streaming chat API with tools
│   ├── api/github/route.ts         # GitHub data source
│   ├── api/posthog/route.ts        # PostHog data source
│   ├── api/slack/route.ts          # Slack data source
│   ├── api/vercel/route.ts         # Vercel data source
│   ├── api/insights/route.ts       # Server-side insights
│   ├── api/telemetry/route.ts      # Telemetry ingest + buffer
│   ├── api/integrations/route.ts   # Integration availability
│   ├── api/memory/feedback/route.ts# Memory feedback
│   ├── page.tsx                    # Entry router to spaces
│   └── spaces/                     # Spaces grid + space pages
├── store/
│   ├── index.ts             # Combined store
│   ├── canvas-slice.ts      # Canvas state
│   ├── workspace-slice.ts   # Spaces, settings, triggers, transforms
│   ├── data-slice.ts        # Data fetching + cache
│   ├── undo-slice.ts        # Snapshot-based undo/redo + batching
│   ├── chat-slice.ts        # Chat messages
│   └── notification-slice.ts# Notifications + polling
├── components/
│   ├── canvas/              # Grid and layout
│   ├── chat/                # Chat UI components
│   ├── spaces/              # Spaces grid + cards
│   ├── notifications/       # Insight notifications
│   ├── tool-ui/             # Tool UI primitives
│   └── ui/                  # Base UI components
├── types/
│   └── index.ts             # All TypeScript interfaces
├── lib/
│   ├── ai-tools.ts          # Tool definitions + system prompt
│   ├── canvas-context.ts    # Serialize canvas + workspace for AI
│   ├── canvas-tools.tsx     # assistant-ui makeAssistantTool definitions
│   ├── canvas-defaults.ts   # Default sizes/bindings
│   ├── component-registry.ts# Component registry
│   ├── templates/           # Template engine + state signals
│   ├── telemetry.ts         # Telemetry writer + buffer
│   ├── telemetry-client.ts  # Client telemetry sender
│   ├── memory/              # Supermemory integration
│   ├── notifications/       # Notification helpers
│   ├── slack-mentions-*.ts  # Slack mention filters/defaults
│   ├── undo/                # Undo types, policies, audit hooks
│   ├── insights/            # Insight types (engine removed)
│   └── audit/               # Audit log implementation
└── hooks/
    └── index.ts             # useCanvas, useHistory, etc.
```

## Testing

Vitest is configured and used for template/undo tests. Future plans:
- React Testing Library for components
- Playwright for E2E

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/petekp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
