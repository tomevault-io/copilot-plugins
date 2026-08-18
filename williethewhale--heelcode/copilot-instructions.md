## heelcode

> - To regenerate the JavaScript SDK, run `./packages/sdk/js/script/build.ts`.

- To regenerate the JavaScript SDK, run `./packages/sdk/js/script/build.ts`.
- The default branch in this repo is `dev`.
- Local `main` ref may not exist; use `dev` or `origin/dev` for diffs.

## Branch Names

Use a short branch name of at most three words, separated by hyphens. Do not use slashes or type prefixes such as `feat/` or `fix/`.

Examples: `session-recovery`, `fix-scroll-state`, `regenerate-sdk`.

## Commits and PR Titles

Use conventional commit-style messages and PR titles: `type(scope): summary`.

Valid types are `feat`, `fix`, `docs`, `chore`, `refactor`, and `test`. Scopes are optional; use the affected package or area when helpful, e.g. `core`, `opencode`, `tui`, `app`, `desktop`, `sdk`, or `plugin`.

Examples: `fix(tui): simplify thinking toggle styling`, `docs: update contributing guide`, `chore(sdk): regenerate types`.

## Style Guide

### General Principles

- Keep things in one function unless composable or reusable
- Do not extract single-use helpers preemptively. Inline the logic at the call site unless the helper is reused, hides a genuinely complex boundary, or has a clear independent name that improves the caller.
- Avoid `try`/`catch` where possible
- Avoid using the `any` type
- Use Bun APIs when possible, like `Bun.file()`
- Rely on type inference when possible; avoid explicit type annotations or interfaces unless necessary for exports or clarity
- Prefer functional array methods (flatMap, filter, map) over for loops; use type guards on filter to maintain type inference downstream
- In `src/config`, follow the existing self-export pattern at the top of the file (for example `export * as ConfigAgent from "./agent"`) when adding a new config module.

Reduce total variable count by inlining when a value is only used once.

```ts
// Good
const journal = await Bun.file(path.join(dir, "journal.json")).json()

// Bad
const journalPath = path.join(dir, "journal.json")
const journal = await Bun.file(journalPath).json()
```

### Destructuring

Avoid unnecessary destructuring. Use dot notation to preserve context.

```ts
// Good
obj.a
obj.b

// Bad
const { a, b } = obj
```

### Imports

- Never alias imports. Do not use `import { foo as bar } from "..."` or renamed imports like `resolve as pathResolve`.
- Never use star imports. Do not use `import * as Foo from "..."` or `import type * as Foo from "..."`.
- If a namespace-style value is needed, import the module's own exported namespace by name, for example `import { Project } from "@opencode-ai/core/project"`, then reference `Project.ID`.
- Prefer dynamic imports for heavy modules that are only needed in selected code paths, especially in startup-sensitive entrypoints. Destructure dynamic import bindings near the top of the narrowest scope that needs them so they read like normal imports. Avoid inline chains such as `await import("./module").then((mod) => mod.value())` or `(await import("./module")).value()`. Keep branch-specific imports inside the branch that needs them to preserve lazy loading.

### Variables

Prefer `const` over `let`. Use ternaries or early returns instead of reassignment.

```ts
// Good
const foo = condition ? 1 : 2

// Bad
let foo
if (condition) foo = 1
else foo = 2
```

### Control Flow

Avoid `else` statements. Prefer early returns.

```ts
// Good
function foo() {
  if (condition) return 1
  return 2
}

// Bad
function foo() {
  if (condition) return 1
  else return 2
}
```

### Complex Logic

When a function has several validation branches or supporting details, make the main function read as the happy path and move supporting details into small helpers below it.

```ts
// Good
export function loadThing(input: unknown) {
  const config = requireConfig(input)
  const metadata = readMetadata(input)
  return createThing({ config, metadata })
}

function requireConfig(input: unknown) {
  ...
}
```

- Keep helpers close to the code they support, below the main export when that improves readability.
- Do not over-abstract simple expressions into many single-use helpers; extract only when it names a real concept like `requireConfig` or `readMetadata`.
- Do not return `Effect` from helpers unless they actually perform effectful work. Synchronous parsing, validation, and option building should stay synchronous.
- Prefer Effect schema helpers such as `Schema.UnknownFromJsonString` and `Schema.decodeUnknownOption` over manual `JSON.parse` wrapped in `Effect.try` when parsing untrusted JSON strings.
- Add comments for non-obvious constraints and surprising behavior, not for obvious assignments or control flow.

### Schema Definitions (Drizzle)

Use snake_case for field names so column names don't need to be redefined as strings.

```ts
// Good
const table = sqliteTable("session", {
  id: text().primaryKey(),
  project_id: text().notNull(),
  created_at: integer().notNull(),
})

// Bad
const table = sqliteTable("session", {
  id: text("id").primaryKey(),
  projectID: text("project_id").notNull(),
  createdAt: integer("created_at").notNull(),
})
```

## Testing

- Avoid mocks as much as possible
- Test actual implementation, do not duplicate logic into tests
- Tests cannot run from repo root (guard: `do-not-run-tests-from-root`); run from package dirs like `packages/opencode`.

## Type Checking

- Always run `bun typecheck` from package directories (e.g., `packages/opencode`), never `tsc` directly.

## PromptLab Inference Boundary

PromptLab inference work must preserve the provider-reasoning path verified against UNC PromptLab on 2026-07-15. Do not replace it with the legacy synthetic tool bridge.

### Verified Transport

- Start a provider turn with `POST /api/agents/chat/:endpoint`, using the authenticated browser-session headers already implemented by `PromptLabClient`.
- Send the user turn in `text` and the harness-owned system/developer instructions in `promptPrefix`. Keep `manualSkills: []` and `ephemeralAgent: false`.
- The start response returns `streamId`, `conversationId`, and `status`. Read the turn from `GET /api/agents/chat/stream/:streamId` as SSE.
- LibreChat commonly uses `event: message` as the outer SSE event and puts the meaningful event name inside the JSON payload's `event` field. Parse both layers.
- Preserve `on_reasoning_delta` separately from `on_message_delta`. Reasoning deltas have been observed as `{ event: "on_reasoning_delta", data: { id, delta } }`; final messages can contain ordered `{ type: "think", think }` and `{ type: "text", text }` parts.
- Continue a conversation with the same `conversationId` and the previous assistant `responseMessage.messageId` as `parentMessageId`. PromptLab currently rebuilds provider history from its own persisted message graph.
- LibreChat v0.8.7 implements this route by synthesizing an internal ephemeral Agent even when the browser payload says `ephemeralAgent: false`. `promptPrefix` becomes that Agent's instructions. HeelCode does not create or configure a PromptLab Agent, and its caller-owned tool list remains empty, but do not describe this route as bypassing LibreChat's Agent implementation.

The implemented harness transport is `POST /v1/native/inference` on loopback `heelcode-promptlabd`. It makes one `PromptLabClient.chat(...)` request, consumes PromptLab SSE directly, and emits canonical HeelCode events. PromptLab models must select this native runtime path and fail closed rather than fall back to `/v1/chat/completions`, the AI SDK compatibility adapter, XML extraction, or generic prose-to-tool inference.

Provider controls verified live:

- GPT through `azureOpenAI`: `useResponsesApi: true`, `reasoning_effort`, `reasoning_summary`, `verbosity`, and `max_tokens`. `gpt-5.4-mini` produced streamed reasoning and a persisted `think` part with `reasoning_summary: "detailed"`.
- Claude through `bedrock`: `thinking: true`, `thinkingBudget`, `effort`, and `maxOutputTokens`. `us.anthropic.claude-sonnet-4-6` produced streamed reasoning and a persisted `think` part.
- Gemini through `google`: `thinking: true`, `thinkingLevel`, and `maxOutputTokens`. `gemini-3.1-pro-preview` produced streamed reasoning and a persisted `think` part.

Model availability changes. Discover current model IDs from PromptLab instead of treating the verified examples as a permanent catalog.

### Non-Negotiable Harness Rules

- Keep model inference on the provider endpoint path above and keep orchestration in Heelcode. PromptLab Agents, Assistants, Actions, MCP tools, web search, and server-side tools are not Heelcode harness primitives.
- Do not use `<heelcode_tool_call>` XML, generic prose-to-tool inference, deterministic tool preflight, or PromptLab-native tool interception for the new harness integration. Those are legacy compatibility mechanisms only. The native parser may recover one explicitly typed trailing `heelcode.tool` object; that is structured action parsing, not inference from prose.
- Do not send OpenAI `messages`, `instructions`, `additional_instructions`, `tools`, or `tool_choice` to the web-chat route and assume they reach the provider. Live canary tests showed that only `text` and `promptPrefix` reached the model; the other fields, including a uniquely named function tool, were stripped.
- Keep tool schemas, tool choice, permissions, execution, results, loops, goals, and subagents inside Heelcode. A PromptLab turn should be one model-inference step in Heelcode's loop, not a delegated PromptLab agent run.
- Caller-owned native function schemas are unavailable through the authenticated student surface tested against LibreChat v0.8.7. Compact provider schemas strip the fields, and the remote Open Responses route builds tools from a saved Agent rather than request `tools`. Do not relabel HeelCode structured actions as native provider tool calls.
- The implemented fallback is typed structured action selection. Prefer a bare `{"type":"heelcode.tool","name":"...","arguments":{...}}` or the explicit `HEELCODE_ACTION` envelope. Because GPT sometimes prepends a progress sentence, HeelCode may extract one unambiguous trailing object beginning exactly with `{"type":"heelcode.tool"`; it still parses the complete object, requires exactly `type`, `name`, and `arguments`, rejects multiple candidates, extra keys, malformed JSON, unknown names, and schema-invalid arguments, creates the call ID locally, and executes through its permission/tool runtime. Ordinary prose without that explicit typed object is never interpreted as a tool call.
- Treat a progress-only response on a turn that still requires an action as action nonconformance, not successful task progress. The Session loop marks that assistant finish `unknown` and appends a private action-conformance user turn, bounded to three attempts and reset after a real tool call. Apply the same explicit-turn recovery to rejected multiple/malformed typed actions after clearing only that protocol error. Never execute a tool inferred from progress prose, blindly replay a semantic protocol failure, or fall back to PromptLab/OpenAI-compatible tool calling.
- The PromptLab harness allowlist is `glob`, `grep`, `read`, `edit`, `write`, and `bash`. Schema validation, permissions, abort propagation, and tool settlement remain HeelCode-owned. Adding any other tool requires an explicit safety review.
- For PromptLab harness turns, the directory passed to the HeelCode invocation is the task workspace boundary even when it is nested inside a broader Git worktree. Prompt instructions must identify that directory as the task workspace, empty directories must be built in place, and local tools must reject parent or sibling paths through `external_directory` before execution. Do not let `--dangerously-skip-permissions` weaken this boundary.
- Do not route this stream through the current `transformPromptLabSSEToOpenAI` compatibility transformer when implementing reasoning support. It handles text and legacy tool calls but drops `on_reasoning_delta`. Parse the native PromptLab event stream into Heelcode's canonical reasoning, text, usage, and completion events.
- Preserve one explicit provider inference request per harness model turn. Do not add a second hidden model/tool loop inside the connector.
- Do not claim this is an unmediated Azure, Bedrock, or Gemini REST API. The request still crosses LibreChat's authenticated chat controller, normalization, persistence, and SSE envelope. "Raw reasoning" here means the structured reasoning content the provider elects to expose, not private model state.
- Do not assume reasoning can be replayed independently yet. The observed final `think` parts did not include Bedrock thinking signatures or Gemini thought signatures. Until a signature-preserving caller-owned history path is verified, reuse PromptLab conversation/message IDs for continuation.
- Preserve empty caller-selected PromptLab tool state and verify `toolSchemaTokens: 0` plus absence of tool-call events in live probes. GPT Responses context bookkeeping reported `toolCount: 1` with an empty tool-token map and 4,224 cached input tokens; sending `useWebSearch: false` did not change that and the flag was not persisted. Do not overstate the current path as proof that LibreChat performs zero internal framing or tool bookkeeping.
- Key PromptLab continuation and active-turn ownership by inference scope, not bare Session ID. Primary work uses `${sessionID}:primary`; advisory/small-model work uses a unique `${sessionID}:advisory:${uuid}` scope so title generation cannot conflict with the primary turn. Keep provider conversation IDs inside the daemon. If that process-local continuation is lost, reconstruct the next request from durable HeelCode Session messages; compaction remains deferred.
- Reject any PromptLab `tool_call`, `tool_calls`, or `tool_use` event on the native transport. It is a provider error, never a HeelCode tool request.
- Treat PromptLab readiness as an authenticated live check. Probe uncached `/promptlab/active` before accepting the cached `/v1/models` catalog; cached model data is not evidence that the bearer or browser cookie is usable.
- Disable Bun's per-request idle timeout for `/v1/native/inference`; provider reasoning can remain quiet longer than Bun's default ten seconds. Independently bound meaningful provider silence with `HEELCODE_PROMPTLAB_SILENCE_TIMEOUT_MS` (default 120 seconds). Only reasoning deltas, message content, and a final response reset that watchdog; SSE heartbeats and unknown status events do not. On expiry, cancel the reader, call PromptLab's abort route, emit one deterministic provider error, and free the inference scope. The processor may retry transient PromptLab silence, loopback 500s, and connection failures at most three times with normal retry status/backoff; semantic action failures require the explicit correction turn above. Never restore an unbounded reconnect loop.
- Treat daemon compatibility as a protocol handshake, not merely a successful `/health` status. Startup must replace a loopback `heelcode-promptlabd` whose protocol does not match the current native transport. If the default loopback connection disappears during a turn, restart the daemon once and retry that request once; do not apply this recovery to custom base URLs, aborted requests, or repeated failures.
- On HTTP 401, allow one bearer refresh and then one recovery from an already authenticated normal Chrome profile, persisting successful replacement session material. PromptLab rotates its refresh cookie in `Set-Cookie`; merge and persist that cookie during both refresh and Chrome capture instead of retaining the now-invalid cookie used for the exchange. If Chrome is logged out, stop before the TUI and leave ONYEN/password/MFA entry to the user. Preserve 401 through the loopback connector and surface an actionable authentication error; never relabel `jwt expired` as inference 500.

### Required Regression Checks

Before replacing or extending the PromptLab transport, verify at least:

- nonempty `on_reasoning_delta` streams for one current GPT reasoning model and Claude Sonnet;
- Gemini thinking coverage when changing Google handling;
- final response content preserves `think` and `text` as separate ordered parts;
- no PromptLab tool-call events for a no-tools reasoning probe;
- a canary present only in `promptPrefix` is visible while canaries in `messages`, `instructions`, `additional_instructions`, and `tools` are absent;
- a two-turn continuation remembers the first turn when reusing `conversationId` and the prior response message ID;
- no bearer tokens, cookies, user identifiers, prompt content, or raw captured streams are committed.
- a real cloned-repository task using multiple `read`/search/edit/write/bash actions, failed-action recovery, test-failure recovery, and a final answer through the structured-action path;
- do not accept read-only calls, a fixed number of provider turns, or a chat-like final response as proof of a working harness. Repository acceptance requires a substantive multi-file implementation, HeelCode-owned tool execution, at least one observed failure-and-recovery loop, the repository's real tests, an independent post-run audit, and a final answer from HeelCode;
- record action-conformance failures, retries, model/quota failures, and any manual steering used during a benchmark. A benchmark that needs explicit action re-prompts can prove the transport and loop while still failing unattended-reliability acceptance;
- malformed arguments and unknown names settle as local tool errors without execution;
- aborting active inference frees the Session for a clean subsequent turn.
- a cold start with expired stored auth cannot pass readiness from a cached catalog; an authenticated Chrome profile recovers automatically, while a logged-out profile stops before model execution with an authentication prompt;

## V2 Session Core

- Keep durable prompt admission separate from model execution. `SessionV2.prompt(...)` admits one durable `session_input` row before scheduling advisory `SessionExecution.wake(sessionID)` unless `resume: false` requests admit-only behavior. The serialized runner promotes admitted inputs into visible user messages at safe boundaries.
- Reusing a Session ID adopts the existing Session. Reusing a prompt message ID reconciles an exact retry only when Session, prompt, and delivery mode match; conflicting reuse fails. Historical projected prompts lazily synthesize promoted inbox records during exact retry.
- Keep `SessionExecution` process-global and Session-ID based. Its local implementation owns the process-local Session coordinator and discovers placement through `SessionStore` plus `LocationServiceMap.get(session.location)` only when a drain starts; no layer should take a Session ID. V2 interruption targets the active process-local ownership chain for that Session; idle or missing interruption is a no-op.
- Keep `SessionRunner`, model resolution, tool registry, permissions, and filesystem Location-scoped. Omitted `Location.workspaceID` means implicit-local placement; explicit workspace identity remains reserved for future placement semantics.
- Preserve one explicit `llm.stream(request)` call per provider turn and reload projected history before durable continuation. Do not bridge through legacy `SessionPrompt.loop(...)` or delegate orchestration to an in-memory tool loop.
- Keep local Session drains process-local until clustering is implemented. `SessionRunCoordinator` joins explicit same-Session resumes, coalesces prompt wakeups, and allows different Sessions to run concurrently. Advisory wakes drain eligible durable inbox rows only; post-crash activity recovery requires a separate explicit design before it may retry provider work.
- Keep delivery vocabulary explicit. Prompts steer by default and coalesce into the active activity at the next safe provider-turn boundary. Explicit `queue` inputs open FIFO future activities one at a time after the active activity settles.
- Keep EventV2 replay owner claims separate from clustered Session execution ownership.
- Keep the System Context algebra, registry, and built-ins in `src/system-context`; keep Context Source producers with their observed domains, and keep Session History selection plus Context Epoch persistence Session-owned.

---
> Source: [WillieTheWhale/heelcode](https://github.com/WillieTheWhale/heelcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
