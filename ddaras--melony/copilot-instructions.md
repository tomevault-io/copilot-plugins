## melony-node-plugin

> Guide for creating Melony plugins in the node app


# Melony Node Plugins

Melony is a minimalist, tiny, and unopinionated agent framework. The core runtime provides a fluent builder API to register event handlers and interceptors.

## Core Runtime Concepts

- **Builder**: Created via `melony<TState, TEvent>()`.
- **Handlers**: Registered via `app.on(eventType, handler)`. Handlers are async generators that yield events.
- **Interceptors**: Registered via `app.intercept(interceptor)` or `app.intercept(eventType, interceptor)`.
- **Plugins**: A function that receives the builder: `(builder: MelonyBuilder<TState, TEvent>) => void`.

## Plugin Structure

When creating a new plugin in `apps/node`, use the `createAppPlugin` helper from `services/plugin-helper.js`:

```typescript
import { createAppPlugin } from '../../services/plugin-helper.js';
import { AppEvent } from '../../types.js';

export const myPlugin = () => createAppPlugin('my-plugin', (builder) => {
  builder.on('some-event', async function* (event, context) {
    // Logic here
    yield { type: 'response-event', data: { ... } } satisfies AppEvent;
  });
});
```

## Types for apps/node

Always use the following types from `apps/node/src/types.ts`:

- `AppState`: `{ threadId: string, runId: string, sessionId: string, orchestration?: OrchestrationState }`
- `AppEvent`: `{ type: string, data?: Record<string, unknown>, meta?: Record<string, unknown> }`

## Usage

In `apps/node/src/runtime.ts`:

```typescript
import { myPlugin } from './plugins/my/plugin.js';
app.use(myPlugin());
```

## HTTP Event Contract (Node app)

- **Single endpoint source of truth**: `@melony/server-node` exposes one route (default `/`) and all client interactions should go there.
- **Always POST events**: send `Content-Type: application/json` and use the JSON body as the Melony event object itself (for example `{ "type": "run", "data": { ... } }`).
- **Model features as event types**: avoid separate REST endpoints for features; instead add new `event.type` values and handle them via `app.on(...)` or plugins.
- **Streaming response**: the server responds with SSE (`text/event-stream`) where each yielded Melony event is sent as `data: <json>`.

## Orchestration Plugin Pattern

- Build agent orchestration as small plugins with one responsibility each (policy, memory, planner, executor, worker agents, observability).
- Keep coordination event-driven (`orchestrator:start -> orchestrator:plan -> orchestrator:plan-ready -> orchestrator:execute -> orchestrator:execution-complete -> orchestrator:complete`).
- For long-running and reconnectable flows, use `background-run` + `watch-run` events instead of additional HTTP routes.

## Worker LLM (`agent:invoke`)

- Orchestration workers call `completeLlmTask` in `plugins/orchestration/llm-worker.ts` (plain `fetch`, no extra deps).
- Set **`OPENAI_API_KEY`** and optionally `OPENAI_MODEL` / `OPENAI_BASE_URL`, or **`GEMINI_API_KEY`** / **`GOOGLE_API_KEY`** and optionally `GEMINI_MODEL` / `GEMINI_BASE_URL`.
- Optional **`MELO_LLM_PROVIDER`**: `openai` or `gemini` to force a backend when multiple keys exist.
- With no key, workers emit a short stub result so the pipeline still runs locally.

---
> Source: [ddaras/melony](https://github.com/ddaras/melony) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
