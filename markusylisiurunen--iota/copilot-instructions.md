## iota

> Multi-provider LLM streaming library supporting OpenAI, Anthropic, and Google. Provides a unified interface for streaming completions with tool calling, reasoning summaries (extended thinking), and token usage with cost tracking.

# Iota

Multi-provider LLM streaming library supporting OpenAI, Anthropic, and Google. Provides a unified interface for streaming completions with tool calling, reasoning summaries (extended thinking), and token usage with cost tracking.

## Architecture

The library is organized around a streaming pipeline that normalizes cross-provider differences behind a unified event interface.

`stream()` in `src/stream.ts` is the main entry point. It first normalizes the conversation context via `normalizeContextForTarget()`, making message history portable across providers by stripping provider-specific metadata and merging system messages. It then validates any tool definitions against a strict JSON Schema subset that all providers support. Finally, it dispatches to a provider-specific implementation based on the model: `streamOpenAI()`, `streamAnthropic()`, or `streamGoogle()`.

Each provider adapter translates the normalized context into the provider's native API format, initiates a streaming request, and processes incoming chunks. Providers don't push events directly to the stream. Instead, they use a `StreamController` which maintains the in-progress `AssistantMessage` state and emits structured events (`part_start`, `part_delta`, `part_end`, etc.) to an `AssistantStream`.

Consumers iterate over the `AssistantStream` using `for await...of` to receive events as they arrive. When the stream completes, calling `result()` returns the final `AssistantMessage` containing the full response content, stop reason, and token usage with cost.

## Key modules

**`src/stream.ts`**: Public entry point exposing `stream()`, `agent()`, and `complete()`. Handles context normalization, tool validation, and provider dispatch.

**`src/assistant-stream.ts`**: Specialized `EventStream` for assistant responses. Completes on `done`/`error` events and exposes `result()` for the final message.

**`src/stream-controller.ts`**: Stateful builder that constructs `AssistantMessage` incrementally. Providers call `addPart()`, `delta()`, `endPart()`, etc.

**`src/event-stream.ts`**: Generic async-iterable queue implementing the producer-consumer pattern with a final result promise.

**`src/models.ts`**: Model registry with capabilities, pricing, and `calculateCost()`.

**`src/types.ts`**: All type definitions for messages, parts, events, and options.

**`src/providers/openai.ts`**: OpenAI Responses API adapter.

**`src/providers/anthropic.ts`**: Anthropic Messages API adapter.

**`src/providers/google.ts`**: Google Gemini API adapter.

**`src/usage.ts`**: Helper for creating empty `Usage` objects.

**`src/utils/json.ts`**: Safe JSON parsing for partial streaming data.

**`src/utils/sanitize.ts`**: Fixes invalid unicode surrogate pairs to prevent SDK failures.

**`src/utils/exhaustive.ts`**: Compile-time exhaustiveness checking helper for switch statements.

## Type system

The codebase uses discriminated unions extensively. Switch on the discriminator and use `exhaustive()` in the default case.

### Messages

Discriminated by `role`:

- **`system`** (`SystemMessage`): Contains `content: string`.
- **`user`** (`UserMessage`): Contains `content: string`.
- **`tool`** (`ToolMessage`): Contains `toolCallId`, `toolName`, `content`, and optional `isError`.
- **`assistant`** (`AssistantMessage`): Contains `provider`, `model`, `content: AssistantPart[]`, `stopReason`, and `usage`.

### AssistantPart

Discriminated by `type`:

- **`text`**: Standard text output with `text: string`.
- **`thinking`**: Reasoning/extended thinking content with `text: string`.
- **`tool_call`**: Tool invocation with `id`, `name`, and `args: unknown`.

All parts have optional `meta` for provider-specific round-trip data.

### AssistantStreamEvent

Discriminated by `type`:

- **`start`**: Stream began, provides initial draft.
- **`part_start`**: New part started at `index`.
- **`part_delta`**: Incremental text chunk for part at `index`.
- **`part_end`**: Part at `index` is complete.
- **`done`**: Stream finished with final `AssistantMessage`.
- **`error`**: Stream failed with partial message and `errorMessage`.

### AgentStreamEvent

For the `agent()` loop, discriminated by `type`:

- **`turn_start`**: New model turn beginning.
- **`assistant_event`**: Proxied `AssistantStreamEvent` from the current turn.
- **`tool_result`**: Tool execution completed with `ToolMessage`.
- **`done`**: Agent loop finished successfully.
- **`error`**: Agent loop failed.

## Streaming system

### EventStream

Generic async-iterable queue in `src/event-stream.ts`. The `push(event)` method enqueues an event, resolving immediately if a consumer is waiting. The `end(result?)` method signals completion and resolves all waiting consumers. Implements `[Symbol.asyncIterator]` to yield from the queue, blocking when empty. The `result()` method returns a promise that resolves when the stream completes.

### AssistantStream

Extends `EventStream<AssistantStreamEvent, AssistantMessage>`. Completes automatically on `type: "done"` or `type: "error"` events. The `resultOrThrow()` method throws if `stopReason` is `"error"` or `"aborted"`.

### StreamController

Stateful builder used by providers in `src/stream-controller.ts`. Exposes `start()` to emit the initial event, `addPart(part)` to add content parts and return their index, `delta(index, text)` for partial text updates, and `endPart(index)` to mark parts complete. Also provides `setUsage(usage)` to update token counts and calculate cost, `setStopReason(reason)` for termination reason, `finish()` to emit `done` and close the stream, and `fail(error)` to emit `error` and close.

## Context normalization

Normalization in `src/stream.ts` makes conversation history portable across providers.

**System messages**: `context.system` plus any `role: "system"` messages are merged into a single system string, joined by `\n\n`. System messages are removed from the message array.

**Assistant messages**: If a message's `{ provider, model }` differs from the target, `thinking` parts are stripped (not portable across providers) while `text` and `tool_call` parts are preserved. Content is normalized to an `AssistantPart[]` array.

**Tool messages**: Always preserved. `isError` defaults to `false`.

**Empty messages**: Filtered out (empty text, assistant with no content).

**Orphaned tool calls**: If an assistant message contains tool calls without matching tool results before the next user or assistant message, synthetic error tool messages are injected with `content: "No result provided"` and `isError: true`. This prevents provider API errors from malformed conversation history.

## Tool schema validation

Validation in `src/stream.ts` ensures cross-provider compatibility.

**Tool names**: Must match `/^[a-zA-Z0-9_-]{1,64}$/`, no duplicates allowed.

**JSON Schema**: Root must have `type: "object"` with `properties`. Allowed keywords are `type`, `description`, `properties`, `required`, `enum`, `minimum`, `maximum`, `items`, and `additionalProperties`. Rejected keywords include `$ref`, `definitions`, `$defs`, `oneOf`, `anyOf`, `allOf`, `pattern`, `format`, and other constraints.

`minimum`/`maximum` are supported for `type: "number"` and `type: "integer"` schemas.

This is a deliberate compatibility subset. All providers support these keywords; advanced JSON Schema features are not portable.

## Provider implementations

### OpenAI (`src/providers/openai.ts`)

Uses the **Responses API** (`client.responses.create`), not Chat Completions. Reasoning is configured via the `reasoning.effort` parameter, with reasoning items stored in `meta` for round-tripping. Tool call arguments are accumulated during streaming and parsed on `output_item.done`. All text is sanitized through `sanitizeSurrogates()`. Key events handled: `response.output_item.added`, `response.output_text.delta`, `response.function_call_arguments.delta`, `response.completed`.

### Anthropic (`src/providers/anthropic.ts`)

Uses Messages API with beta headers for streaming features. The `sanitizeToolCallId()` function replaces non-alphanumeric characters with `_` to meet Anthropic's ID requirements. Thinking blocks require a `signature` for round-tripping, stored in `meta`. Reasoning budget is calculated via `anthropicBudget()` based on effort level and max tokens. Beta headers used: `interleaved-thinking-2025-05-14`, `fine-grained-tool-streaming-2025-05-14`. Key events handled: `message_start`, `content_block_start`, `content_block_delta`, `content_block_stop`, `message_delta`.

### Google (`src/providers/google.ts`)

Uses `@google/genai` streaming. Reasoning is enabled via `thinkingConfig.includeThoughts`. The `thoughtSignature` is stored in `meta` for round-tripping. Tool call IDs are generated if not provided by the API.

## Agent loop

`agent()` in `src/stream.ts` implements a multi-turn tool execution loop, returning an `AgentStream`.

The loop emits `turn_start`, calls `stream()`, and proxies all `AssistantStreamEvent` as `assistant_event`. On completion: if tool calls are present, it executes the corresponding handlers, emits `tool_result` for each, and continues the loop. If no tool calls are present, it emits `done` and terminates. On error or abort, it emits `error` and terminates.

Termination conditions: no tool calls in response (task complete), `maxTurns` exceeded, `stopReason` is `"error"` or `"aborted"`, or an unknown tool is called (no handler).

Message history accumulates across turns. Each `AssistantMessage` and `ToolMessage` is appended for subsequent turns.

## Options

`StreamOptions` for `stream()`:

- **`apiKey`** (`string?`): Provider API key. Falls back to `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, or `GOOGLE_API_KEY` environment variables.
- **`temperature`** (`number?`): Sampling temperature.
- **`maxTokens`** (`number?`): Max output tokens. Defaults to `model.maxOutputTokens`, capped at 65536.
- **`reasoning`** (`ReasoningEffort?`): One of `none`, `minimal`, `low`, `medium`, `high`, `xhigh`. Clamped to model capabilities.
- **`serviceTier`** (`ServiceTier?`): OpenAI only. One of `flex`, `standard`, `priority`. Affects cost calculation: flex = 0.5x, standard = 1x, priority = 2x.
- **`signal`** (`AbortSignal?`): Cancel the request. Sets `stopReason: "aborted"`.

`AgentOptions` extends `StreamOptions` with `maxTurns` to limit the agent loop.

## Debug logging

Set `IOTA_DEBUG_LOG_DIR` to a directory path to write near-raw request/response payloads as a pretty-printed JSON file with a provider prefix and timestamp (for example `openai-2025-01-01T12-00-00-000Z.json`).

## Error handling

**Pre-stream validation errors**: Thrown synchronously for missing API keys, unsupported tools, or invalid schemas.

**Streaming errors**: Captured by providers and emitted as `type: "error"` events. The stream still completes with a partial `AssistantMessage` where `stopReason: "error"`.

**Result access**: The `result()` method returns `AssistantMessage` even on error; check `stopReason`. The `resultOrThrow()` method throws an `Error` if `stopReason` is `"error"` or `"aborted"`, with the partial message attached as `.assistantMessage`.

**Abort**: Pass an `AbortSignal` via options. The provider cancels the request and `stopReason` becomes `"aborted"`.

## Model registry

`src/models.ts` defines supported models with `provider`, `id`, `name`, `contextWindow`, and `maxOutputTokens`. The `supports` object tracks capabilities: `reasoning`, `tools`, and `reasoningXhigh`. The `pricing` object contains rates per 1M tokens for `input`, `output`, `cacheRead`, and `cacheWrite`.

`calculateCost(model, usage)` computes a cost breakdown from token counts.

## Public API

Exported from `src/index.ts`:

**Functions**: `stream`, `agent`, `complete`, `completeOrThrow`, `calculateCost`, `getModel`, `getApiKey`, `clampReasoning`, `clampReasoningForModel`, `supportsXhigh`, `normalizeContextForTarget`.

**Classes**: `AssistantStream`, `AgentStream`.

**Data**: `openaiModels`, `anthropicModels`, `googleModels`.

**Types**: All from `src/types.ts` and `src/models.ts`.

## Development

```bash
npm run check    # Biome format + typecheck
npm run build    # ESM + CJS to dist/
npm test         # Unit tests (no API calls)
npm run smoke    # E2E tests (requires API keys)
```

**Tests**: Unit tests are named `tests/*.test.ts` (no `e2e-` prefix) and test validation, normalization, and internal logic without API calls. E2E tests are named `tests/e2e-*.test.ts`, hit real provider APIs, and require `IOTA_SMOKE=1` plus provider API keys. Do not set `temperature` in tests; use provider defaults.

**Style**: Biome with 2-space indent and 100 line width. Types use `PascalCase`, values and functions use `camelCase`, files use `lowercase.ts`.

**TypeScript**: Prefer exhaustive `switch` for discriminated unions. Use `default: return exhaustive(value)` so missing cases become compile-time errors.

## Releasing

Publishing to npm happens automatically via GitHub Actions when a GitHub Release is published.

1. Ensure you are on `main` with a clean working tree.
2. Verify and build:
   ```bash
   npm run check
   npm run build
   ```
3. Bump the version and create a tag:
   ```bash
   npm version patch|minor|major
   ```
4. Push the commit and tag:
   ```bash
   git push --follow-tags
   ```
5. Create a GitHub Release (triggers publish):
   ```bash
   gh release create v$(node -p "require('./package.json').version") --generate-notes
   ```

## Maintaining this file

Keep AGENTS.md in sync with the codebase. When changes affect architecture, exports, provider behavior, type definitions, or schema support, update the relevant sections here.

---
> Source: [markusylisiurunen/iota](https://github.com/markusylisiurunen/iota) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
