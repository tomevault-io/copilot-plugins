## featherpress

> OpenAI-compatible proxy for [Featherless.ai](https://featherless.ai). Preconfigured with a single API key (`rc_...`) in `.config/config.json` and a hardcoded 3-model catalog. Built to survive Featherless's 32k context ceiling via opencode's native subagents (`explore`, `general`) plus the mind MCP for durable memory, with a FIFO queue enforcing the 1-concurrent-call cap (see "Session Memory Protocol" below).

# Featherless-Proxy — Developer Guide

OpenAI-compatible proxy for [Featherless.ai](https://featherless.ai). Preconfigured with a single API key (`rc_...`) in `.config/config.json` and a hardcoded 3-model catalog. Built to survive Featherless's 32k context ceiling via opencode's native subagents (`explore`, `general`) plus the mind MCP for durable memory, with a FIFO queue enforcing the 1-concurrent-call cap (see "Session Memory Protocol" below).

## Project Structure

```
FRESHH/
├── proxy.js              # Main proxy implementation + session endpoints (main lane only)
├── dashboard.html        # Dashboard (cache stats, key pool, test chat, wallpapers)
├── .config/
│   └── config.json       # Runtime configuration (API key, hardcoded models, wallpapers, etc.)
├── .cache/
│   ├── sessions/         # Main session log: main.jsonl (subagents use mind MCP, not the proxy)
│   ├── i18n/             # Cached UI translations per locale
│   ├── wallpaper*.jpg    # Cached wallpapers (Bing / Wallhaven / FreeGen)
│   └── usage.db          # (legacy, currently unused — Featherless has no app-usage API)
├── .logs/                # HTTP error logs
├── package.json
├── start.cmd             # Auto-detect launcher (Bun preferred, Node fallback)
├── start-node.cmd        # Node-only launcher
├── skills.md             # Opencode provider configuration reference
├── README.md             # User documentation
└── AGENTS.md             # This file
```

## Constants & Config

- `FEATHERLESS_API_BASE` — `https://api.featherless.ai/v1`
- `API_KEY_ENV_VAR` — `FEATHERLESS_API_KEY`
- `HARDCODED_MODELS` — 3 entries: `moonshotai/Kimi-K3`, `deepseek-ai/DeepSeek-V4-Flash-0731`, `zai-org/GLM-5.2`
- `I18N_TRANSLATE_MODEL` — `zai-org/GLM-5.2` (used for dashboard autotranslation)
- `loadConfig()` — Loads `.config/config.json` with env var overrides
- `saveConfig()` / `debouncedSaveConfig()` — Writes config (debounced 500ms)

## Key Components

### 1. Hardcoded Model Catalog

The model list is hardcoded in `proxy.js` (`HARDCODED_MODELS`). Each entry has `id`, `name`, `reasoning`, `context`, `output`, `tool_call`, `vision`. The proxy does NOT call Featherless's `/v1/models` to populate the catalog at boot — it only calls it to validate the API key (`validateApiKey()`).

`/v1/models` returns the hardcoded list in OpenAI format. `setupOpencodeConfig()` writes all 3 `HARDCODED_MODELS` entries into every discovered `opencode.json`. Subagents use opencode's native `explore`/`general` types (no special model ids) and write findings to mind MCP directly.

### 2. Shell-Tool Guard

`isGitCommand(cmd)` detects shell commands that invoke `git` and rewrites matched tool_calls to `echo "BLOCKED: git commands are disabled by proxy policy"`. Applies to non-streaming responses, streaming SSE responses (buffered and re-emitted), and cached responses on cache hit.

### 3. Retry Logic

`retryLoop(fn)` retries the upstream `/v1/chat/completions` request up to `MAX_RETRIES` (10) with escalating delays (3s, 6s, 9s, …). Retries on HTTP 500, 503, and network failures. On each retry the current key is marked unhealthy so the key pool rotates. Non-retryable HTTP errors are returned immediately.

### 4. Response Cache

LRU cache for non-streaming LLM responses. Key: MD5 of `(model + stream_flag + system + messages + tools)`. TTL default 60s, max size 100. Stats via `GET /api/cache`; clear via `DELETE /api/cache`. Cached responses are passed through the shell-tool guard before being returned.

### 5. Key Pool

Round-robin multi-key pool with cooldown/unhealthy marking. Used when `config.keys` has multiple entries; otherwise the single `config.apiKey` is used. `acquire()` returns `{ key, name, index }` and sets `config.apiKey` + `upstream.apiKey`. `markUnhealthy(index, status)` cools down 60s (503), 30s (502), or 10s (other).

### 6. Upstream Client

`UpstreamClient` with `getModels()` (GET `/v1/models`, 10s timeout) and `chatCompletions(body)` (POST `/v1/chat/completions`). Uses `UPSTREAM_AGENT` — keep-alive HTTPS agent (128 sockets, 60s keepalive). Sends `HTTP-Referer: https://featherless.ai/` and `X-Title: Featherless Proxy` headers as Featherless requests for client attribution.

### 7. Tool Schema Normalization

Normalizes JSON Schema in tools to handle `$ref`, `$defs`, `definitions`, nullable patterns. Applied to `payload.tools` when any tool has `$ref`/`$defs`/`$definitions` in its parameters.

### 8. Stream/Body Utilities

`readBodyText(body)` handles Node streams, Web ReadableStreams, and async iterables. `pipeBodyToResponse(body, res)` pipes with abort handling. SSE streams are buffered through `sanitizeSseResponse()` so the shell-tool guard can intercept tool_call deltas.

### 9. Sleev Context-Compression Gateway (optional)

When `SLEEV_ENABLED=true`, the proxy spawns the Sleev gateway on `127.0.0.1:17321` and points the opencode provider at Sleev instead of the proxy directly. Topology: `opencode → Sleev (compress) → Featherless-Proxy → Featherless`. Sleev provides transparent lossy context compression, which buys time between manual digest cycles (see Session Memory Protocol). Disable Sleev if you want full control over what gets compressed.

### 10. FreeGen / Bing / Wallhaven Wallpapers

Unrelated to Featherless. The dashboard can rotate background images from Bing (peapix), Wallhaven, or AI-generated by FreeGen. The current wallpaper is embedded as base64 in the dashboard HTML `<head>` to prevent white flash on load.

### 11. i18n Autotranslation

The dashboard UI can be autotranslated into any locale via `I18N_TRANSLATE_MODEL` (default `zai-org/GLM-5.2`). Translations are cached in `.cache/i18n/<locale>.json`. If `LOCALE` is set in config, that locale is forced. If no API key is configured, the dashboard stays in English.

## Session Memory Protocol — Surviving the 32k Context Ceiling

Featherless caps context at ~32k. Opencode's normal compaction is lossy and is disabled when Sleev is on. To do real work (multi-file refactors, long research) you split working memory across multiple contexts using **opencode's native subagents** (`explore`, `general`) plus the **mind MCP** for durable storage.

### The Tier Model

| Tier | Tool | Lifetime | Lossy? | Holds |
|---|---|---|---|---|
| Hot | main agent context (≤24k working headroom) | single turn | no | current task, last ~5 tool calls |
| Warm | Sleev compression buffer (optional) | session | yes | recent turns, distilled |
| Cold | mind MCP (`projects/<repo>`) | forever | no | decisions, root causes, patterns, reference docs, pitfalls |
| Rescue | `/api/session/digest` + `/api/session/reset` (main only) | session | yes | structured summary of main's recent turns when context fills |

Sleev and mind don't overlap: Sleev is the short-term compressor, mind is the durable truth. The proxy's only session-memory job is **main's own context rescue** — when main's 32k fills, the proxy can summarize main's recent turns so main can continue with the digest + a fresh context. Subagents don't need proxy-side tracking; they write to mind directly.

### Subagents (use opencode's native Task tool)

Spawn subagents via the Task tool with `subagent_type`. Two types are available:

- **`explore`** — fast, read-only codebase exploration (file patterns, content search, "how does X work?"). Use for any read that would consume >2k of main's context.
- **`general`** — general-purpose multi-step work (write code, run tests, multi-file research). Use when a subtask needs agency.

Subagents have their own 32k context and their own tool access. If the mind MCP server is configured for the project, subagents can call `memory_add` / `memory_query` / `checkpoint_*` directly — that's how they file their findings to the durable store.

**Role guidance** (just prompt content — not special model ids):
- "Read these files and return a ≤2k digest of their public API" → `explore`
- "Find everywhere the auth middleware is used and return file:line refs" → `explore`
- "Implement the FOO feature in src/bar.ts and run the tests" → `general`
- "Review this diff against the style guide and return file:line issues" → `general`

### Concurrency Discipline (Featherless 1-unit cap, enforced by proxy)

The proxy defaults to `OVERRIDE_CONCURRENCY: 1` and FIFO-queues anything above that. This is the hard Featherless plan limit: **only ONE upstream API call may be in flight at any instant.** The proxy enforces it; you don't have to count.

Implications for the agent:

- **NEVER spawn subagents in parallel.** Even though opencode's Task tool supports parallel subagents via multiple tool calls in one message, doing so just piles them into the FIFO queue — they'll run strictly one after another, and the wait blocks the earlier subagent's return. Spawn subagents **sequentially** (one Task call, wait for the result, then the next Task call).
- Opencode's Task tool is synchronous — main's LLM call ends when it emits the Task call, so main does NOT consume a slot while a subagent runs. Peak concurrency with sequential subagents = 1 (the running subagent). This fits the plan exactly.
- Nested subagents (subagent spawning a subagent) work the same way — they queue at the proxy and run one at a time. Keep nesting shallow (depth ≤ 2) because each level adds a round-trip through the queue.
- The `/api/session/digest` endpoint also calls the upstream — it counts as 1 concurrent call. **Only call digest when no subagent is running.** Check `/api/concurrency` first: if `active > 0`, wait. Otherwise the digest call will 429 against an in-flight subagent.
- Queue timeout is 120s. If a request sits in the FIFO queue longer than that, the proxy returns HTTP 503 `queue_full` to the caller. With sequential subagents this should never trigger; if it does, it means you have a stuck subagent — abort it and retry.

Practical pattern:
1. Spawn one subagent (one Task call).
2. Wait for its result.
3. Optionally call `/api/concurrency` to confirm `active == 0`.
4. Spawn the next subagent, or call digest, or continue main.

### Session Endpoints (main only)

| Method | Path | Purpose |
|---|---|---|
| `GET`  | `/api/session/status` | Main log size (entries + bytes) |
| `GET`  | `/api/session/log?lane=main&limit=<n>` | Raw transcript entries for main |
| `GET`  | `/api/session/digest?lane=main&model=<id>&max_entries=<n>` | Builds a transcript from the main log and asks `model` (default `deepseek-ai/DeepSeek-V4-Flash-0731`) to produce a structured summary: `GOAL / DONE / PENDING / BLOCKERS / KEY_FILES / DECISIONS / NEXT_ACTION` |
| `POST` | `/api/session/reset?lane=main` | Clears main's session log. Conversation affinity ages out via LRU. |
| `POST` | `/api/session/append` | Manually append a synthetic entry (e.g. a digest receipt) to main's log. Body: `{ lane, model, request, response }` |

### The Protocol

```
1. BOOT (session start):
   - checkpoint_query → restore last checkpoint
   - memory_query(space, "current task keywords") → load cold tier

2. PER SUBTASK:
   - delegate heavy reads → Task(subagent_type="explore", prompt="…")
     · subagent returns a digest; main never reads big files directly
   - delegate multi-step work → Task(subagent_type="general", prompt="…")
     · subagent does the work and writes results to mind via memory_add
   - on verified result → memory_add (cold tier) + checkpoint_save

3. CONTEXT PRESSURE (main approaching ~24k):
   - first check /api/concurrency — if active > 0, a subagent is still
     running; wait for it to finish before calling digest (digest is an
     upstream call and counts against the 1-concurrent cap)
   - curl http://localhost:8084/api/session/digest?lane=main
     → returns structured digest of main's recent turns
   - main files the digest to mind itself via memory_add
     (cat:summary, T3) and calls checkpoint_save
   - curl -X POST http://localhost:8084/api/session/reset?lane=main
   - continue main with: original goal + last digest + next action only

4. SESSION END:
   - checkpoint_done → final session-* memory
   - anything crossing the durable threshold → memory_add with proper cat tag
```

### Promotion to Durable Memory (before digesting)

Digesting is lossy. Before running `/api/session/digest`, promote anything exact to mind:

- decisions → `memory_add` `cat:decision`
- root causes → `memory_add` `cat:bugfix`
- patterns → `memory_add` `cat:pattern`
- config facts → `memory_add` `cat:config`
- pitfalls → `memory_add` `cat:discovery` + update `known-pitfalls` living reference

Then digest the prose around them. Never rely on the digest to preserve exact paths, code, or decisions — those go to mind first, always.

### Model Choice

The 3 hardcoded models cover different points on the cost/reasoning curve:
- `deepseek-ai/DeepSeek-V4-Flash-0731` — cheap + fast. Use for digests, simple subagent work, i18n.
- `zai-org/GLM-5.2` — balanced. Default for main agent and most subagent work.
- `moonshotai/Kimi-K3` — strong reasoning + vision. Use for hard problems, image input, critic-style review.

The proxy advertises all 3 in `opencode.json` under the `featherless` provider. Pick the model that matches the task's difficulty; don't burn Kimi-K3 on a digest that DeepSeek Flash can do.

## API Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/healthz` | Proxy health check |
| `GET` | `/v1/models` | OpenAI-format model list (3 hardcoded models) |
| `POST` | `/v1/chat/completions` | OpenAI-format chat completions |
| `GET` | `/api/config` | Get proxy configuration (key masked) |
| `POST` | `/api/config` | Update proxy configuration |
| `GET` | `/api/validate` | Validate API key against Featherless `/v1/models` |
| `GET` | `/api/models` | List enabled models + display names |
| `GET` | `/api/keys` | List API keys (masked) |
| `POST` | `/api/keys` | Add/update/delete API keys |
| `GET` | `/api/cache` | Cache stats |
| `DELETE` | `/api/cache` | Clear cache |
| `GET` | `/api/usage` | Local in-memory usage snapshot (per-session counters) |
| `GET` | `/api/usage-history` | Empty 90-day scaffold (Featherless has no app-usage API) |
| `GET` | `/api/concurrency` | Active + queued requests, configured limit |
| `GET` | `/api/session/status` | Main session log size |
| `GET` | `/api/session/log` | Raw main transcript |
| `GET` | `/api/session/digest` | Structured summary of main's transcript |
| `POST` | `/api/session/reset` | Clear main's transcript log |
| `POST` | `/api/session/append` | Append a synthetic entry to main's log |
| `GET` | `/api/bg` | Bing wallpaper proxy |
| `GET` | `/api/bg-wallhaven` | Wallhaven wallpaper proxy |
| `GET/POST` | `/api/bg-freegen` | FreeGen AI wallpaper generator |
| `GET` | `/api/i18n` | Translation bundle for the dashboard |
| `POST` | `/api/restart` | Restart the proxy (exit code 42, launcher re-spawns) |
| `GET/POST` | `/api/sleev` | Sleev gateway status / toggle |

The `/v1/messages` (Anthropic) endpoint is intentionally absent — Featherless is OpenAI-compatible only.

## Configuration

`.config/config.json`:

| Field | Description | Default |
|---|---|---|
| `LISTEN_ADDR` | Proxy listen address | `127.0.0.1:8084` |
| `UPSTREAM_BASE_URL` | Featherless API URL | `https://api.featherless.ai/v1` |
| `API_KEY` | Featherless API key | `rc_...` (preconfigured) |
| `REQUEST_TIMEOUT` | Upstream request timeout | `15m` |
| `CACHE_TTL` | Response cache TTL | `60s` |
| `CACHE_MAX_SIZE` | Max cached responses | `100` |
| `CACHE_ENABLED` | Enable/disable cache | `true` |
| `ENABLED_MODELS` | Reserved (catalog is hardcoded in proxy.js) | `[]` |
| `MODEL_DISPLAY_NAMES` | Custom display names per model | `{...}` |
| `API_KEYS` | Array of allowed proxy API keys (auth) | `[]` |
| `OVERRIDE_CONCURRENCY` | Cap on in-flight upstream requests (0 = uncapped) | `0` |
| `MAX_IMAGES` | Max image attachments per forwarded request | `9` |
| `AUTO_ROTATE_THRESHOLD` | Estimated-token threshold that triggers auto context-rotation of the main lane (0 = disabled). When the main lane log exceeds this, the proxy digests it, resets the log, and prepends the digest as a system message to the next outgoing request. | `12000` |
| `AUTO_COMPRESS_ENABLED` | When `true`, the outgoing-request size guard attempts to digest older messages and retry before falling back to the 400 `context_length_exceeded_proxy_guard` error. When `false`, the guard just returns 400. | `true` |
| `DISABLED_MODELS` | Hardcoded model ids to hide from `/v1/models` | `[]` |
| `wallpaperSource` | `none` / `bing` / `wallhaven` / `freegen` | `freegen` |
| `FREEGEN_PROMPT` | Default prompt for FreeGen wallpaper | `epic cinematic landscape, …` |
| `LOCALE` | Force dashboard locale | — |
| `SLEEV_ENABLED` | Spawn Sleev gateway for context compression | `false` |

## Startup Sequence

1. `loadConfig()` — Load `.config/config.json` + env var overrides
2. `ResponseCache` — Init with config values
3. `KeyPool` — Init from `config.keys` or single default key
4. `UpstreamClient` — Init HTTP client
5. `validateApiKey()` — Hit Featherless `/v1/models` to verify the key
6. `startSleev()` — If `SLEEV_ENABLED`, spawn Sleev gateway
7. `http.createServer(handleRequest).listen(port, '127.0.0.1')` — Start HTTP server on port 8084
8. `setupOpencodeConfig()` — Discover + write all hardcoded × lane model entries to every `opencode.json`. Creates `openconfig.b4featherless.json` backup before first edit.
9. Auto-open dashboard on first run

## Testing

```bash
node --check proxy.js
node proxy.js
curl http://localhost:8084/healthz
curl http://localhost:8084/v1/models
curl http://localhost:8084/api/session/status
curl "http://localhost:8084/api/session/digest?lane=main"
curl -X POST "http://localhost:8084/api/session/reset?lane=main"
curl -X POST http://localhost:8084/v1/chat/completions \
  -H "Authorization: Bearer rc_..." \
  -H "Content-Type: application/json" \
  -d '{"model":"zai-org/GLM-5.2","messages":[{"role":"user","content":"hi"}]}'
```

## Dependencies

Zero external npm dependencies — uses only Node.js built-in modules (`fs`, `path`, `os`, `http`, `https`, `crypto`, `child_process`), native `fetch` (Node 18+), and native `WebSocket` (Node 24+) for FreeGen.

## Notes for Opencode Agents

When working on Featherless-Proxy through opencode, keep the following in mind.

### Edit tool / exact replacements

The `edit` tool requires an exact text match for `oldString`. If you get `Could not find oldString in the file`, read the file first, copy the exact block (including whitespace, indentation, line endings), and paste verbatim. Use `replaceAll: true` only when you intend to replace every occurrence.

### Web research

Use `webfetch` to retrieve content from a known URL. Do not call `websearch` (not available unless explicitly enabled). Prefer `webfetch` for Featherless docs (https://featherless.ai/docs/).

### Provider configuration

- The proxy auto-writes a `featherless` provider into every discovered `opencode.json`.
- The generated config sets `"instructions": ["AGENTS.md", "skills.md"]` so this guide and the provider reference are loaded on startup.
- After the proxy updates `opencode.json`, restart opencode for the changes to take effect.
- The proxy advertises 15 models per `opencode.json`: 3 base models × 5 lanes (main + 4 subagents). Pick the lane-suffixed variant when spawning a subagent so its traffic is logged to the right lane.

### Session memory protocol (when working on long tasks)

Follow the protocol in the "Session Memory Protocol" section above. Specifically:

- For any subtask that involves reading more than ~2k of file content, delegate to an `explore` subagent via the Task tool instead of reading files in main.
- When main's context gets crowded (≈24k), check `/api/concurrency` (`active == 0`), then run `/api/session/digest?lane=main`, file the digest to mind yourself via `memory_add` (cat:summary), then `/api/session/reset?lane=main` before continuing.
- Before digesting any lane, promote exact facts (paths, decisions, code refs) to mind via `memory_add` with the appropriate `cat:*` tag — digesting is lossy, mind is not.
- Spawn subagents **strictly sequentially** — one Task call, wait for the result, then the next. Never spawn in parallel. The Featherless plan allows only 1 concurrent API call; the proxy FIFO-queues anything above that, so parallel spawns just waste time in the queue.
- Before calling `/api/session/digest`, check `/api/concurrency` — if `active > 0`, wait. Digest is an upstream call and counts against the 1-concurrent cap.

## Auto Context Rotation

The proxy automatically rolls the main lane over into a fresh 32k session when the accumulated main lane log approaches the ceiling — no agent-side changes required. The flow:

1. At the top of `proxyChatRequest`, before the upstream call goes out, the proxy estimates the main lane's token total (`estimateLaneTokens`, ~4 chars/token).
2. If the estimate exceeds `AUTO_ROTATE_THRESHOLD` (default 12000, leaving ~20k headroom for the new turn + digest output + large pastes), the proxy:
   - Builds a transcript from the main lane log (`buildTranscriptFromLaneLog`).
   - Calls the digest model upstream (default `deepseek-ai/DeepSeek-V4-Flash-0731`) using the same prompt shape as `/api/session/digest` (GOAL / DONE / PENDING / BLOCKERS / KEY_FILES / DECISIONS / NEXT_ACTION).
   - Appends the digest as a synthetic lane log entry with `request: "digest"` (so `estimateLaneTokens` skips it), clears the lane log, then re-appends the digest entry as the sole record.
   - Prepends the digest as a `system` message to the OUTGOING request's `messages` array (inserted as the first message, before any user-supplied system prompt): `[Session rotated by proxy — previous 32k context digested]\n\n<digest text>`.
3. The agent's next turn then carries the recovered context.

### Config field

`AUTO_ROTATE_THRESHOLD` (default `12000`, env var `AUTO_ROTATE_THRESHOLD`). Set to `0` to disable. Read in `loadConfig`, exposed via `GET /api/config`, persisted via `POST /api/config`, serialized by `saveConfig`. Lowered from 20000 → 12000 to cooperate with the outgoing-request guard: the proxy now checks the OUTGOING request size AFTER rotation, so rotation must leave more headroom for the current turn. 12000 leaves 20k headroom (vs 12k before), giving the outgoing-request guard space to digest the current turn's older messages if needed.

### Outgoing request guard with cooperate (rotate-then-compress)

Auto-rotation only fires on the ACCUMULATED lane LOG. A single outgoing request can itself exceed the 32k ceiling (e.g. the agent pasted a huge file into one message) — at that point the lane log may be small, so auto-rotation never triggers on its own, and Featherless rejects with `Maximum context length ... exceeds by N tokens`.

To close that gap, the proxy COOPERATES the two mechanisms: rotation first, compression second. The flow runs at the top of `proxyChatRequest`, AFTER the existing auto-rotation block but BEFORE the upstream call, gated on `lane === 'main'` and `AUTO_COMPRESS_ENABLED !== false` (the flag gates the ENTIRE cooperate flow, not just compression):

1. **Estimate the outgoing request payload tokens** (`requestTokens`): sum of `estimateTokens` over every message's content (string content directly; array/vision content JSON-stringified), plus `estimateTokens(JSON.stringify(tools || []))`. Look up the target model's context limit via `getModelInfo` (lane suffix already stripped; defaults to 32000 if not in `HARDCODED_MODELS`).
2. If `requestTokens > modelContextLimit - 4000` (4k headroom reserved for the response + digest overhead — was 2k, now 4k to be safer), run the cooperate sequence:
   - **Step A — Try rotation.** If `laneLogSize('main').entries > 0`, run the existing auto-rotation flow (build transcript from lane log, call digest model via `upstream.chatCompletions` directly, append digest entry to log, clear log). Then prepend the digest as a `system` message to the OUTGOING request's `messages` array (inserted as the FIRST message). Re-estimate `requestTokens`. If now under budget, proceed with the upstream call — DONE.
   - **Step B — Try auto-compress.** If still over budget (or the lane log was empty), and the request has 3+ messages, split into: `preservedSystem` (first message if `role=system`), `oldMessages` (all except the last 2 — or last 1 if only 3 total after peeling system), `recentMessages` (the last 2, or last 1). Digest `oldMessages` via `upstream.chatCompletions` directly. Build `[preservedSystem?, compressedDigestSystemMsg, ...recentMessages]`. Re-estimate. If now under budget, proceed — DONE.
   - **Step C — Fall back to HTTP 400.** If STILL over budget (or compression was impossible — fewer than 3 messages after peeling system), return HTTP 400 `context_length_exceeded_proxy_guard`. The proxy genuinely cannot help at this point — the recent messages alone are too big.

The 400 fallback message:

```
code: "context_length_exceeded_proxy_guard"
message: "Request payload (~NNNNN tokens) exceeds the 32k context budget for <model>.
          The proxy cannot split a single request across sessions. Delegate this
          task to a subagent via opencode's Task tool (subagent_type='general') so
          the work is broken into ≤24k chunks across multiple 32k sessions. Tip:
          pass only a digest of large files to the subagent, not the full content."
```

This is the proxy's way of forcing subagent delegation for oversized single requests it cannot compress: the proxy cannot spawn opencode subagents itself, but it can refuse the request and tell the agent to do so. The agent should respond to this error by spawning a `general` subagent via the Task tool with a smaller prompt — e.g. a digest of the large file instead of the full content.

The cooperate block runs AFTER auto-rotation, so if the lane log was the problem, auto-rotation already handled it. The cooperate block only fires when the REQUEST ITSELF is too big (or auto-rotation didn't fully solve it). All digest calls use `upstream.chatCompletions` directly (bypassing `processQueue`) because `proxyChatRequest` already holds the concurrency slot — re-entering the queue would deadlock.

### Auto-compress config field

`AUTO_COMPRESS_ENABLED` (default `true`, env var `AUTO_COMPRESS_ENABLED`). When `true`, the ENTIRE cooperate flow runs (rotation + compression). When `false`, the proxy skips ALL auto-management of the outgoing request and just sends it as-is — Featherless will reject if too big. This is the escape hatch for debugging. Read in `loadConfig`, exposed via `GET /api/config`, persisted via `POST /api/config`, serialized by `saveConfig`.

### Scope and safety

- **Main lane only.** Subagent lanes are NOT auto-rotated — they use the mind MCP for durable storage per the Session Memory Protocol and must not be touched by the proxy. The check is gated on `lane === 'main'` (via `parseLane`).
- **Non-fatal on digest failure.** If the digest upstream call fails, the proxy logs the error via `logHttpError` and continues with the original request unchanged — the user's request is never blocked by a digest failure.
- **Loop-break guarantee.** After a rotation, the lane log contains exactly ONE entry: the synthetic digest entry, which is marked `request: "digest"` and skipped by `estimateLaneTokens`. The next call's estimate is therefore 0, so no rotation triggers. The agent's actual new turn then gets appended normally via the existing logging path.
- **Concurrency-safe.** The digest call uses `upstream.chatCompletions` directly (bypassing `processQueue`) because `proxyChatRequest` already holds the concurrency slot acquired by `handleChatCompletions`/`processQueue`. Re-entering the queue would deadlock trying to re-acquire the slot.

---
> Source: [notBlubbll/featherpress](https://github.com/notBlubbll/featherpress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
