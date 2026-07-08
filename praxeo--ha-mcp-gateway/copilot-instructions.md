## ha-mcp-gateway

> Guidance for Claude Code working in this repository. This file is comprehensive

# CLAUDE.md

Guidance for Claude Code working in this repository. This file is comprehensive
and self-contained; `README.md` is the longer narrative reference (it opens with
the design history). When the two overlap, the code is authoritative — verify
before relying on either.

---

## What this project is

**HA-MCP-Gateway** is a Cloudflare Worker + Durable Object that bridges a Home
Assistant smart home to LLMs. It:

- holds a persistent WebSocket to Home Assistant and mirrors live entity state
  in memory,
- streams every meaningful HA event into a queryable D1 **forensic log**,
- exposes the whole surface as an **MCP server** (`POST /mcp`) for clients like
  Claude Desktop and Claude Code,
- runs a built-in **chat agent** — "Ranger," GLM 5.2 Fast on Fireworks
  (runtime-configurable; see LLM configuration) with reasoning enabled and a
  native tool-calling loop — reachable at a web
  `/chat` UI and via the `ai_chat` MCP tool,
- short-circuits deterministic cover (garage/bay door) commands via a sub-500ms
  fast path that skips the LLM entirely.

It is a single-household production deployment — the author's home. There is no
staging environment, and a push to `main` deploys to the live house (see
Build, test, deploy below). Treat changes with appropriate care.

The autonomous "heartbeat" agent that once ran every 60 seconds has been
removed — see the Story section in `README.md`. There is exactly one execution
path now: user-driven chat.

---

## Build, test, deploy

Working directory: `C:\Users\obert\ha-mcp-gateway`. Shell is **PowerShell on
Windows**.

```powershell
npm install          # dependencies (only devDependency is vitest)
npm test             # vitest run — 6 suites (forensic filter, light sanitizer,
                     #   scheduler, fast path, coalesce, llm-config)
```

- **Deploy is git-driven.** A push to `main` triggers a **Cloudflare Workers
  Builds** pipeline (set up in the Cloudflare dashboard, so it leaves no
  artifact in the repo) that runs the build and deploys. A merged commit goes
  live — treat `git push` to `main` as a production deploy.
- `wrangler deploy` from the repo root is the manual / local alternative; it
  runs the same build.
- **Build** (either path) runs the `wrangler.toml` `[build]` directive —
  `npx esbuild src/worker.js --bundle --outdir=dist --format=esm --outbase=src`
  — producing `dist/worker.js`.
- **`dist/worker.js` is a build artifact. Never edit it by hand** — it is
  overwritten on every build. All source lives in `src/`.
- A deploy reconciles bindings, cron triggers, and Durable Object migrations
  with Cloudflare.
- Tests run with `vitest` — 6 suites under `test/` (98 cases): the forensic
  noise filter (`should-log-state-change`), the light-service sanitizer, the
  scheduler, the cover fast path, the flat-arg coalescer, and the runtime
  LLM-config helpers (`llm-config`). Add tests alongside these when changing
  pure, testable logic; the pure helpers keep a synced stub copy in their test.

---

## Critical gotchas — read before changing code

1. **Durable Object isolate staleness.** The persistent HA WebSocket keeps the
   DO's V8 isolate resident across deploys — Cloudflare never hibernates it, so
   it never reloads new code on an ordinary deploy. **Worker-side changes
   (`src/worker.js`) take effect immediately. DO-side changes
   (`src/ha-websocket.js`) do not.** To force a fresh DO isolate you must rename
   the DO class via a `renamed_classes` migration in `wrangler.toml` and update
   `class_name` in the `durable_objects.bindings`. The class is currently
   `HAWebSocketV29` — it has been renamed 28 times for exactly this reason.
   Procedure for any DO-side change you need live:
   - bump the class name (`HAWebSocketV29` → `HAWebSocketV30`) in the `export
     class` line in `src/ha-websocket.js`, every `HAWebSocketV29.` static
     reference, the `export {}` at the file end, and the `worker.js` import;
   - add a `[[migrations]]` block with `tag = "v30"` and
     `renamed_classes = [{ from = "HAWebSocketV29", to = "HAWebSocketV30" }]`;
   - update `class_name` in `[[durable_objects.bindings]]`.
   **Exception — the LLM config is now runtime-configurable** (V29): the chat/
   agent model, endpoint, and reasoning mode can be swapped live via the
   `/admin/llm-config` route (DO storage override) with no rename or redeploy.
   See the LLM configuration section. A model swap no longer needs this dance.
   DO storage (snapshot, chat history, memory, cursors) is preserved across
   renames. Diagnose staleness with `wrangler tail --format json` (compare
   per-event `scriptVersion.id` to the latest deploy) or by comparing
   `/admin/version` (Worker) against the DO `/version` route.

2. **Never edit `dist/`.** It is generated.

3. **Pooling discipline.** Every Workers AI embedding call must use
   `pooling: "cls"`. The `ha-knowledge` Vectorize index was built with cls
   pooling; mismatched pooling between backfill and query produces near-random
   rankings.

4. **Cloudflare Access fronts the Worker.** A direct `curl` /
   `Invoke-RestMethod` hits the Access login page, not the app. Use
   `cloudflared access curl` or a service token for any HTTP probe.

5. **PowerShell quoting.** To POST JSON, write the body to a UTF-8-**without-BOM**
   temp file and use `--data-binary "@file"`. Inline JSON on the PowerShell
   command line gets mangled. See the smoke-test snippets in `README.md`.

6. **Production is live.** There is no staging. A deploy — including a `git
   push` to `main` — changes the behavior of a real, occupied house. Don't
   push speculative changes to `main`.

7. **Commit hygiene.** Commit messages in this repo follow
   `type(scope): VNN — summary` (e.g. `feat(do): V29 — runtime LLM config + Qwen 3.7 Plus default`). The
   `VNN` matches the DO migration tag when the change is DO-side.

---

## Architecture

```
User (web /chat · Claude Desktop · Claude Code · WhatsApp[dormant])
        │
        ▼
Cloudflare Worker  (src/worker.js)
   │   HTTP routing, MCP JSON-RPC handler (82-tool surface), embedded
   │   CHAT_HTML UI, ElevenLabs STT proxy, multi-kind knowledge backfill,
   │   scheduled() cron handler
   ▼
Durable Object  HAWebSocketV29  (src/ha-websocket.js)
   │   singleton "ha-websocket-singleton" — persistent HA WebSocket,
   │   in-memory stateCache, hibernation snapshot, chat tool loop,
   │   cover fast path, forensic D1 writers, reconnect backfill
   ├─► Home Assistant WebSocket (JWT auth) — subscribes to state_changed,
   │     entity_registry_updated, device_registry_updated,
   │     automation_triggered, call_service
   ├─► D1 "ha_db"            — ai_log, observations, state_changes,
   │                           automation_runs, service_calls
   ├─► Vectorize "ha-knowledge" — nine-kind semantic index
   ├─► Workers AI            — @cf/baai/bge-large-en-v1.5 embeddings (cls)
   ├─► Fireworks             — GLM 5.2 Fast chat completions (runtime-configurable)
   └─► ElevenLabs Scribe     — speech-to-text
```

The Worker is stateless request routing. All long-lived state and all LLM calls
live in the single Durable Object. The DO is a singleton — every request
addresses the same instance by the fixed name `ha-websocket-singleton`.

---

## Repo layout

| Path | Role |
|---|---|
| `src/worker.js` | Worker entry. MCP handler (`TOOLS` — 82 entries — + `handleTool` dispatch, `getAgentToolset`, `DANGEROUS_TOOLS`), HTTP routes, embedded `CHAT_HTML`, ElevenLabs STT proxy, KV cache helpers, per-kind `build*Docs`, `backfillKnowledge`, `scheduled()` cron handler. |
| `src/ha-websocket.js` | The Durable Object class `HAWebSocketV29`. Persistent HA WS, `stateCache`, chat path (`chatWithAgentNative`), native tool loop (`runNativeToolLoop`), tool dispatch (`executeNativeTool`), action executor (`executeAIAction`), prompt builders, fast path, forensic D1 writers, reconnect backfill, `alarm()` keepalive. `_sanitizeLightServiceData` strips unsupported color descriptors from `light.turn_on` / `light.toggle` calls before they reach HA. ~6000 lines — the bulk of the system. |
| `src/agent-tools.js` | OpenAI-format tool schemas for the chat agent: `NATIVE_AGENT_TOOLS` (19 = 6 action + 13 read), `CHAT_ALLOWED_TOOL_NAMES`, `NATIVE_ACTION_TOOL_NAMES`. |
| `src/vectorize-schema.js` | Shared Vectorize schema + helpers: `vectorIdFor`, `topicTagFor`, `fnv1aHex`, per-kind embed-text builders, `isNoisyEntity` / `isNoisySwitch` / `isNoisyService`, `buildMetadata`. Imported by both `worker.js` and `ha-websocket.js`. |
| `migrations/000{1,2,3}_*.sql` | D1 schema. 0001 indexes legacy tables; 0002 creates the forensic log tables; 0003 de-dupes `state_changes` and adds the backfill-idempotency unique index. |
| `wrangler.toml` | Bindings, `[build]`, cron triggers, DO migrations v1→v22. |
| `dist/worker.js` | esbuild output — **build artifact, never edit**. |
| `BUGS.md` | The iteration backlog — bugs captured via `report_bug`, folded in at iteration time. |
| `.dev.vars` | Local-dev secrets — never committed. |

---

## The two tool surfaces

These are distinct and should not be conflated:

- **MCP surface** — `TOOLS` in `src/worker.js`, **82 tools**, returned by the
  MCP `tools/list` method. Full HA control (entities, devices, areas, floors,
  labels, automations, scripts, scenes, dashboards, calendars, todos, input
  helpers, services) plus agent-state tools, the three forensic query tools,
  the three ephemeral scheduler tools (`schedule_action`,
  `list_scheduled_actions`, `cancel_scheduled_action`), `report_bug`, and
  `ai_chat`. `getAgentToolset` filters out `DANGEROUS_TOOLS` (12 tools —
  `restart_ha`, automation create/update/delete, bulk enable/disable, etc.)
  for non-external roles; external MCP clients get the full set. `handleTool`
  is the dispatcher.
- **Chat agent native surface** — `NATIVE_AGENT_TOOLS` in `src/agent-tools.js`,
  **19 tools**, the curated set the in-house chat agent gets. 6 action tools
  (`call_service`, `ai_send_notification`, `save_memory`, `save_observation`,
  `schedule_action`, `cancel_scheduled_action`) and 13 read tools (`get_state`,
  `get_logbook`, `render_template`, `vector_search`, `get_house_topology`,
  `get_automation_config`, `report_bug`, `query_state_history`,
  `query_automation_runs`, `query_causal_chain`, `get_nws_weather`,
  `get_nws_discussion`, `list_scheduled_actions`). Dispatched by
  `executeNativeTool`.

The forensic query tools and several others share single helper implementations
between the two surfaces — when changing a shared tool, check both call sites.

---

## Key subsystems

### Forensic event log

Every `state_changed` (filtered by `_shouldLogStateChange`), every
`automation_triggered`, and every `call_service` HA event is written
fire-and-forget to D1 tables `state_changes` / `automation_runs` /
`service_calls`, with HA's context-chain IDs preserved. 90-day rolling
retention. On WS reconnect, `_backfillStateChangesFromHA` pulls missed state
changes from HA's history API (idempotent via `INSERT OR IGNORE` on a unique
index). Surfaced to the agent through `query_state_history`,
`query_automation_runs`, and `query_causal_chain`. `_shouldLogStateChange` is
the noise filter (Zigbee LQI/RSSI suffixes, monotonic counters, numeric
deadbands) — it has unit-test coverage; update the test when you change it.

### Knowledge index (`ha-knowledge`)

Cloudflare Vectorize, 1024-dim, cosine, `@cf/baai/bge-large-en-v1.5` with cls
pooling. Nine kinds: entity, automation, script, scene, area, device, service,
memory, observation. Entity/device refresh on HA registry events; memory/
observation are write-through; automation/script/area/device/service resync
nightly. `vector_search` (both surfaces) does metadata-filtered semantic search.
Schema and helpers are in `src/vectorize-schema.js` — the single source of truth
for vector IDs and metadata shape.

### HOUSE_STATE_SNAPSHOT

`_buildHouseStateSnapshot()` emits a small, authoritative text block read from
the live `stateCache` — locks, garage/bay covers, thermostats, presence, power,
contact sensors, and the Tesla Model Y. It is **gated**: `buildDynamicContext`
injects it only when the user message matches `HOUSE_STATUS_TRIGGER_RE`, or
vector retrieval found no high-confidence entity, or no message was passed. The
TRUTHFULNESS prompt rule depends on it.

### Native tool loop & chat prompt

`chatWithAgentNative` is the one chat path. It tries the cover fast path first,
then builds context and runs `runNativeToolLoop`: call the model, dispatch
`tool_calls` via `executeNativeTool`, push results, repeat. The chat path uses
`maxIterations: 6`, `maxTokens: 4096`, `hallucinationGuard: true`.
`callLLMWithTools` posts to Fireworks with `temperature: 0`,
`thinking: { type: "enabled" }`, and a 45s `AbortController` timeout.

The system prompt is split for prefix caching (V13):
`getStaticChatSystemPrompt()` is 100% static (byte-identical every request —
caches as a stable prefix) and `buildDynamicContext()` holds all per-request
data, prepended to the trailing user message. **When editing the prompt, keep
anything per-request out of the static half** — interpolating a dynamic value
into `getStaticChatSystemPrompt()` silently destroys the prefix cache.

### Cover fast path

`_tryDeterministicFastPath` pattern-matches imperative garage/basement-bay
open/close commands and fires `cover.open_cover` / `cover.close_cover` directly,
skipping the LLM. It always uses explicit open/close (never `toggle`), no-ops if
already in the target state, and falls through to the LLM on any error or
ambiguity.

### Ephemeral scheduler

`_scheduleAction` / `_listScheduledActions` / `_cancelScheduledAction` /
`_fireDueScheduledActions` on the DO power the three new tools
(`schedule_action`, `list_scheduled_actions`, `cancel_scheduled_action`).
Pending tasks live in DO storage under the `scheduled:<uuid>` namespace; the
60s `alarm()` tick calls `_fireDueScheduledActions` which fires any past-due
tasks. Two delivery rules: **delete-before-fire** (better to lose a task than
fire it twice for non-idempotent services like `toggle`) and a **WS-up guard**
(skip the tick when the HA WS is down so a transient outage doesn't destroy
the task). The compound "on for an hour" / "AC down 2 then back up" patterns
are agent-orchestrated — chat agent calls `call_service` immediately and
`schedule_action` for the reversal in the same turn, snapshotting any
"from current temp" arithmetic at schedule time.

### Crons

Two triggers in `wrangler.toml`:
- `* * * * *` → `prewarmCache` (warms KV; heavy registry refresh every 15 min;
  reconnects the DO if the WS is dead).
- `30 8 * * *` → `dailyKnowledgeResync` + `dailyAiLogRetention` (30-day `ai_log`
  prune) + `dailyForensicLogRetention` (90-day forensic prune).

The DO's `alarm()` keeps the WS alive (ping/pong, reconnect on no-pong) and
also calls `_fireDueScheduledActions` to fire any due scheduled tasks. The
60s reschedule is **mandatory**. Do not remove the reschedule.

---

## LLM configuration

**Runtime-configurable since V29.** The effective config is resolved at call
time by `resolveLLMConfig` (module-level, pure) with three layers, highest
precedence first:

1. **DO storage override** — `state.storage` key `llm_config`, set live via the
   `/admin/llm-config` route (worker) → `/llm_config` (DO). No deploy, no rename.
2. **env vars** — `LLM_ENDPOINT` / `LLM_MODEL` / `LLM_REASONING_MODE` /
   `LLM_REASONING_EFFORT` (a fresh isolate picks these up).
3. **baked-in defaults** — the static class constants in `src/ha-websocket.js`,
   exposed via `HAWebSocketV29._defaultLLMConfig()`:

```js
static LLM_ENDPOINT = "https://api.fireworks.ai/inference/v1/chat/completions";
static LLM_MODEL = "accounts/fireworks/routers/glm-5p2-fast"; // GLM 5.2 Fast
static LLM_REASONING_MODE = "effort";     // "thinking" | "effort" | "none"
static LLM_REASONING_EFFORT = "high";     // used only when mode === "effort"
```

Both call sites (`callLLM`, `callLLMWithTools`) go through `await
this._getLLMConfig()` and use the resolved `cfg.endpoint` / `cfg.model`, so they
can't drift. Reasoning is applied by `applyReasoningToBody(body, cfg)`: mode
`"thinking"` → `thinking: { type: "enabled" }`; `"effort"` →
`reasoning_effort: <effort>`; `"none"` → neither. Fireworks **rejects sending
`thinking` and `reasoning_effort` together**, so the helper always emits at most
one and clears any pre-existing field first. Reasoning still returns in
`reasoning_content`, so the extraction / SSE stream / prior-turn re-feed are
unchanged. The pure helpers have unit coverage in `test/llm-config.test.js`
(keep the stubs there in sync).

Operating the runtime config:

```text
GET  /admin/llm-config                          # effective + stored + env + defaults
POST /admin/llm-config  {"model":"accounts/fireworks/models/minimax-m3",
                         "reasoning_mode":"thinking"}   # set override (merged)
POST /admin/llm-config  {"reset":true}          # clear override → back to env/defaults
```

The override is validated by `sanitizeLLMConfigPatch` (only `endpoint`, `model`,
`reasoning_mode`, `reasoning_effort`; enums enforced). A set updates the
in-process cache, so even the pinned live isolate honors it on the next call.

The API key is `env.FIREWORKS_API_KEY`. Provider history (kept here because it
explains the migration tags): originally MiniMax → gpt-oss-120b on Groq (V13–15)
→ MiniMax again (V16) → DeepSeek V4 Flash on Fireworks (V17–V26) → MiniMax M3 on
Fireworks (V27–V28) → Qwen 3.7 Plus on Fireworks (V29, runtime-configurable) →
GLM 5.2 on Fireworks → GLM 5.2 Fast on Fireworks (still V29 class — a model swap
is exempt from the DO-rename/migration dance per gotcha #1; the default change
ships in code and goes live via the `/admin/llm-config` override or a fresh
isolate). GLM 5.2 Fast is the `accounts/fireworks/routers/glm-5p2-fast`
high-speed serving path — the same GLM 5.2 model and quality at a higher
per-token price. It drives reasoning via `reasoning_effort`
(mode `"effort"`), not the Anthropic-style `thinking` toggle Qwen used — its
`low`/`medium`/`high` all map to the High tier; only `max`/`xhigh` reach Max
(outside our effort enum). Don't reintroduce Groq or the old MiniMax endpoint/auth
(`MINIMAX_API_KEY`, `reasoning_split`). The current default is GLM 5.2 Fast, served by Fireworks via
`FIREWORKS_API_KEY`; any OpenAI-compatible Fireworks model can be selected at
runtime via the override.

---

## Bindings, secrets, and flags

**Bindings** (`wrangler.toml`): `HA_WS` (DO `HAWebSocketV29`), `HA_CACHE` (KV),
`KNOWLEDGE` (Vectorize `ha-knowledge`), `AI` (Workers AI), `DB` (D1 `ha_db`),
`CF_VERSION_METADATA`.

**Secrets** (`wrangler secret put`): `HA_URL`, `HA_TOKEN`, `FIREWORKS_API_KEY`,
`ELEVENLABS_API_KEY` (optional, for `/transcribe`), `MCP_AUTH_TOKEN` (optional).

**Flags**: `USE_NATIVE_TOOL_LOOP` (`"true"` — the chat path),
`USE_VECTOR_ENTITY_RETRIEVAL` (`"true"` — vector context build, flat-list
fallback), `CLIMATE_PREAMBLE_ENABLED` (optional), `DUMP_SYSTEM_PROMPT`
(`"1"` — one-shot preflight dump of the request body to `ai_log`; unset after).

---

## HTTP routes (Worker)

`/health`, `/transcribe` (ElevenLabs STT proxy), `/refresh`, `/chat` (GET = UI,
POST = SSE chat), `/twilio` (dormant), `/mcp` (and `/` — MCP JSON-RPC), plus
admin endpoints: `/admin/bugs`, `/admin/bugs/clear`, `/admin/recent_activity`,
`/admin/token-usage` (GET — per-day chat token totals from `ai_log`; `?days=N`
1–30, `?format=json|markdown`), `/admin/version`,
`/admin/llm-config` (GET/POST — runtime LLM config),
`/admin/index-stats`, `/admin/reindex-observations`,
`/admin/cleanup-stale-vectors`, `/admin/rebuild-knowledge`.

The DO has its own internal HTTP router (`/status`, `/llm_config`, `/reconnect`,
`/version`, `/vector_search`, `/call_service`, the registry routes, the forensic
query routes, etc.) — reached only via `doFetch` from the Worker.

---

## Conventions when editing

- Match the existing code style — plain ES modules, no framework, no TypeScript,
  no build step beyond esbuild bundling.
- LLM endpoint/model/effort live in static class constants; the two call sites
  (`callLLM`, `callLLMWithTools`) must not drift. Change the constant.
- Forensic D1 writes are deliberately fire-and-forget — never make the WS event
  handler `await` a D1 write. Failures increment `_d1WriteFailures`.
- `executeNativeTool` and `handleTool` are the two dispatchers. A new chat tool
  needs: a schema in `agent-tools.js`, a `case` in `executeNativeTool`, and
  (if it mutates state) membership in `NATIVE_ACTION_TOOL_NAMES`.
- The chat system prompt is large and load-bearing. Behavioral rules
  (TRUTHFULNESS, ACTION CONFIRMATION, TOOL ERROR HANDLING, COMMITMENT RULE) are
  there for real past failures — don't trim them casually.
- Timestamps in agent-facing text must be US Central. Tool results pass through
  `_reformatToolResultTimestamps`; the prompt forbids the model emitting ISO/UTC.
- After a DO-side change, you must bump the class name + add a migration (see
  gotcha #1) or the change will not go live.

---

## Bug capture & iteration workflow

Bugs found during use are captured conversationally — the user says "that's a
bug" / "log this as broken" and the chat agent calls `report_bug`, which stores
a structured entry (description + recent chat turns + recent `ai_log` + cited
entity state) in DO storage under per-id keys.

At iteration time: pull `/admin/bugs?format=markdown`, prepend to `BUGS.md`,
commit, then `POST /admin/bugs/clear`. Fix one at a time; after fixing, strike
through the `BUGS.md` heading and add `Fixed: <commit-sha>` — don't delete
entries, the history is the audit trail.

---
> Source: [praxeo/ha-mcp-gateway](https://github.com/praxeo/ha-mcp-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
