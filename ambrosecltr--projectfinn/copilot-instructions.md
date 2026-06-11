## projectfinn

> **Generated:** 2026-05-04

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-05-04
**Commit:** FINN-56 pattern expansion
**Branch:** pattern-expansion-review

## OVERVIEW

Finn is a single-user AI companion that lives in iMessage. Built as a Bun/TypeScript monorepo with a multi-agent architecture: a hot-path agent handles real-time messages, background workers tackle long tasks, and Pattern workers handle scheduled or trigger-driven automations. Runtime state is backed by Postgres (Drizzle ORM) plus Finn's inbuilt profile, conversation, file, worker, connector, and Pattern systems.

## STRUCTURE

```
FinnAI/                        # Git + workspace root
├── AGENTS.md                  # Repo-wide engineering notes for coding agents
├── identity/                  # Personality/voice prompts (FINN.xml, FINN.kids.xml) — runtime injected
├── prompts/                   # Agent process instructions (hot-path, worker, pattern workers, compactor) — first-class runtime artifacts
├── docs/                      # Operational docs (architecture, config, agents, memory, tools, etc.)
├── docker/                    # entrypoint.sh + sandbox.Dockerfile — app boot and worker sandbox image
├── packages/
│   ├── core/                  # Shared types, Zod config, EventBus, logger, errors, tracing
│   ├── db/                    # Drizzle schema + Postgres client (singleton)
│   ├── llm/                   # Provider-agnostic LLM abstraction (Anthropic/OpenAI/Fireworks/DeepSeek/OpenAI-compatible)
│   ├── agents/                # Agent implementations (hot-path, worker, compactor) ← has AGENTS.md
│   ├── tools/                 # Tool definitions for hot-path + worker agents ← has AGENTS.md
│   ├── messaging/             # Spectrum adapter, message routing, sender
│   ├── media/                 # Deepgram STT, ElevenLabs TTS, file storage, attachment processing
│   ├── cron/                  # Shared scheduling helpers
│   ├── patterns/              # Pattern store + scheduler for scheduled/composio automations and run history
│   ├── integrations/          # External clients: Exa/Parallel web, Fal, Composio, BrowserUse, MCP manager
│   ├── toolsets/              # Project-local toolsets with model instructions + allowlisted executors
│   ├── puter/                 # Tauri macOS menu bar app for paired local iMessage/Notes Personal Intelligence
│   ├── web/                   # Vite/React companion app for login, profile, connectors, patterns, and recent pattern runs
│   └── server/                # Hono HTTP server, routes, event wiring — app entry point
├── drizzle.config.ts
├── package.json               # Bun workspace monorepo root
└── tsconfig.json              # Strict TS, ESNext, bundler resolution, @finn/* path aliases
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add/modify hot-path tools | `packages/tools/src/hot-path/` | See tools/AGENTS.md |
| Add/modify worker tools | `packages/tools/src/worker/` | See tools/AGENTS.md |
| Add/modify document or attachment extraction | `packages/media/src/document-extractor.ts` + `packages/media/src/attachment-processor.ts` + `packages/toolsets/src/toolsets/files/` | Keep extraction local-only, bounded, and gated by runtime |
| Change agent behavior | `packages/agents/src/` | See agents/AGENTS.md |
| Modify LLM prompts | `prompts/*.xml` | Treat as code — shapes runtime LLM behavior |
| Change prompt assembly / runtime gating | `packages/agents/src/prompt-factory.ts` + `packages/tools/src/worker/factory.ts` | Capability map controls what prompts/tools expose |
| Change personality/voice | `identity/FINN.xml` | Lowercase always, never split replies |
| Add env vars / config | `packages/core/src/config.ts` | Zod schema + env loader; changes propagate everywhere |
| DB schema changes | `packages/db/src/schema.ts` | Then `bun run db:generate && bun run db:migrate` |
| Add HTTP routes | `packages/server/src/routes/` | Wire in `server/src/index.ts` |
| Change web app UI/behavior | `packages/web/src/` | Main app is in `packages/web/src/main.tsx`; patterns/connectors UI and run history live there |
| Change Puter Mac app behavior | `packages/puter/` | Tauri app; update Rust commands and React UI together |
| Change Puter live bridge behavior | `packages/server/src/puter-bridge.ts` + `packages/server/src/personal-intelligence-service.ts` | Puter PI must use live paired Mac commands, not batch uploads |
| Add/modify project-local toolsets | `packages/toolsets/src/` | Toolset instructions, schemas, and executors used by internal automation like Puter PI |
| Event wiring (worker delivery / Pattern surfacing) | `packages/server/src/event-wiring.ts` | Controls which events persist + deliver to hot-path |
| Memory activity feed events | `packages/server/src/activity-feed.ts` + `packages/integrations/src/memory.ts` | Provider-neutral operational events retained through memory |
| External integrations | `packages/integrations/src/` | MCP, Exa/Parallel web, Fal, Composio, BrowserUse |
| Startup / DI wiring | `packages/server/src/index.ts` | 367-line bootstrap — creates all subsystems |
| User runtime/workspace/Finn JS workspace wiring | `packages/server/src/user-runtime.ts` + `packages/runtime/src/` + `packages/tools/src/code-mode.ts` | Per-user workspace roots, file storage, memory, MCP, process runtime views, and Secure Exec Finn JS workspace tools |

## PACKAGE DEPENDENCY GRAPH

```
core ←── db ←── messaging, media, cron, patterns, agents, runtime
  ↑              ↑
  ├── llm ←───── agents
  ├── integrations ←── tools/runtime/server
  ├── toolsets ←── tools/server
  ├── tools ←──── agents (tools depends on messaging, integrations, media, runtime, toolsets)
  ├── web (client-only companion app)
  ├── puter (Tauri companion app)
  └── server (imports everything; composition root)
```

**@finn/core is the single source of truth** for types, config, errors, logger, EventBus. Changes there ripple everywhere.

## CONVENTIONS

- **Workspace**: Repo root is the Bun workspace root. Run commands from `FinnAI/`.
- **Runtime**: Bun — runs TypeScript directly. No transpile step for dev.
- **Module imports**: `@finn/*` path aliases (tsconfig.json paths). `workspace:*` deps in package.json.
- **Patched dependencies**: `spectrum-ts@1.13.1` is patched via `patches/spectrum-ts@1.13.1.patch`. The patch preserves iMessage text/caption metadata so `MessageRouter` can route mixed text+photo sends correctly. Remove this patch only after Spectrum upstream preserves text on iMessage attachment/group messages and the router caption regression tests pass without it.
- **Server export**: `packages/server/src/index.ts` exports `{ port, fetch }` — Bun's native fetch handler pattern (no `app.listen()`).
- **Single-process workers**: Workers run in-process via WorkerManager — no separate queue/process. CPU-bound work impacts the server.
- **Config pattern**: Zod-validated at startup via `loadConfig()`. Cached singleton. Test helper: `resetConfig()`.
- **ID generation**: `generateId(prefix)` from core/utils (e.g., `generateId("evt")`, `generateId("wrk")`).
- **Logging**: Pino with secret redaction. Pre-built loggers: `hotPathLogger`, `workerLogger`, `compactorLogger`, `autonomyLogger`.
- **Telemetry**: PostHog is the only supported telemetry backend. Telemetry is optional and starts only when `POSTHOG_API_KEY` is set and `TELEMETRY_PROVIDER` is unset or `posthog`; `TELEMETRY_PROVIDER=none` disables it. AI SDK calls should use `createFinnTelemetry()` so PostHog receives Finn-scoped identities (`posthog_distinct_id`, `$ai_session_id`, `tenantId`, `userId`, `processType`, and run ids where relevant), native tool metadata, and token usage through `@posthog/ai`. The OpenTelemetry PostHog span processor is wrapped so only Finn-marked AI traces, plus their child spans, are exported; preserve the Finn span marker/baggage path when adding LLM telemetry.
- **Tracing**: OpenTelemetry optional — `withSpan()` wrapper is zero-overhead when uninstrumented.
- **Tool pattern**: `tool({ description, inputSchema: z.object({...}), execute })` — `tool` imported from `"ai"` (Vercel AI SDK). See tools/AGENTS.md.
- **Capability map**: `config.capabilities` from `packages/core/src/config.ts` is the shared source of truth for optional runtime behavior. If a capability changes, update tool factories, prompt assembly, status visibility, and tests together.
- **Prompt assembly**: Hot-path identity is assembled through `packages/agents/src/prompt-factory.ts`, which gates optional voice, creative, worker, and integration guidance. Worker, pattern-management, and pattern-worker prompts keep baseline behavior only; exact tool-family instructions come from the worker runtime appendix.
- **Hot-path execution**: The hot path now uses streamed generation with explicit `finish_turn` completion so tool execution can happen during generation without relying on implicit model stop behavior.
- **Hot-path prompt boundaries**: Hot-path current-turn, runtime, status, worker, trigger, and auto-memory context should be represented as explicit prompt envelopes. Only `human_message` envelopes are user-authored; runtime/status/memory/worker/trigger envelopes are internal context and should not be allowed to masquerade as a new user turn.
- **Hot-path inline vision**: User-sent image attachments are auto-loaded into current-turn vision only up to the hot-path inline image cap, and each loaded image must go through the shared model-image byte preparation path. Extra images stay in attachment metadata with file IDs/paths so the hot path can delegate or explicitly inspect them without claiming unseen pixels.
- **Hot-path delivery receipts**: Multi-bubble or mixed delivery actions are allowed across hot-path steps when they belong to the same response. Delivery tool results are outbound receipts, not new user messages; the model should continue only for unsent content and then call `finish_turn`.
- **Hot-path memory modes**: `MEMORY_MODE` controls how provider memory enters the hot path. `hybrid` injects capped provider-formatted recall into trailing runtime context and exposes hot-path memory tools, `context` injects capped recall without hot-path memory tools, and `tools` exposes tools without auto injection. Auto recall must go through `MemoryRuntimeService`, stay fail-open, compact, deduped, and must not leak provider internals like Hindsight retain context, IDs, tags, chunk IDs, or metadata blobs. For Hindsight, hot-path auto context should prefer `observation` facts first and only fall back to raw `world` / `experience` facts when no observations match. Auto-recall timeout is controlled by `MEMORY_AUTO_RECALL_TIMEOUT_MS`; auto-recall result count is controlled by `MEMORY_AUTO_RECALL_MAX_RESULTS` (default 8).
- **Memory profile context**: A always-available synthesized `<memory_profile>` envelope (static + dynamic) is built per turn through `MemoryClient.buildProfileContext`. For Hindsight this is backed by Finn-managed mental models (pre-computed reflect responses), not live reflect calls, and must stay fail-open and sanitized (no IDs, tags, or provenance). Supermemory uses its native profile endpoint. The profile-seed (`syncUserProfileSeedToMemory`) writes the authoritative name/location/timezone baseline for both Supermemory and Hindsight.
- **Hindsight bank provisioning**: On first use per process, Finn idempotently configures each user bank's missions, disposition, and entity labels, and (when `MEMORY_PROVISION_MENTAL_MODELS` is true, the default) provisions a fixed set of Finn-managed mental models (auto-refreshed after observation consolidation via `trigger.refresh_after_consolidation`, `mode: "delta"`) plus reflect directives. Provisioning is fail-open and only creates missing Finn-managed models/directives, never touching manual ones. Set `MEMORY_PROVISION_MENTAL_MODELS=false` to disable. The user reflect mission and disposition (skepticism 3) are tuned to synthesize from partial evidence rather than abstaining.
- **Hindsight Personal Intelligence tuning**: User-bank missions are tuned so Finn captures both durable facts and the smaller texture of daily life (interests, habits, places), while only ephemeral moods and routine logistics stay transient. The observations mission enforces canonicalization (one observation per person/topic/constraint, no near-duplicate fragments, constraints consolidated). Entity labels assign one best-fit `memory_domain` (with an `operational` domain for Pattern/operational state so it never reads as personality) and reserve `durability:durable` for stable facts. Retained metadata is limited to a PI allowlist of provenance/scope/dedup IDs (`accountScopeId`, `sourceId`, `messageId`, `threadId` are read back by PI dedup); raw source JSON blobs are dropped. Keep these missions, entity-label vocabularies, and the metadata allowlist aligned when changing memory behavior.
- **Memory runtime boundary**: Raw memory provider clients are app-level integrations only. Feature code and tool factories should consume `UserRuntimeServices.memory` / `MemoryRuntimeService`; user scoping, timezone, recorder construction, and provider calls belong behind the runtime boundary.
- **Memory activity feed**: Operational lifecycle facts should flow through `activity_feed_event` on `EventBus` and the `wireMemoryActivityFeed` subscriber, which resolves the user's memory runtime. Pattern create/edit/pause/resume/delete events are user-bank operational state in Hindsight and must be fail-open so memory outages never block the mutation.
- **Core profile fields**: `users.display_name`, `users.timezone`, `users.location`, and `metadata.profile.timezoneSource` are the durable operational profile. `timezoneSource` is `server`, `browser`, or `manual`; the web dashboard captures browser timezone once when still `server`, and manual wins thereafter. Broader user understanding belongs in memory providers, not new profile columns by default.
- **Hot-path profile completion**: The hot path may receive `update_user_profile` to silently set a missing explicit display name or durable home/base location. It must not use this for travel, workplaces, preferences, weak location hints, or broad personal facts. Missing-name guidance is dynamic runtime context, not a questionnaire.
- **Pattern metadata split**: Saved patterns separate user-facing copy from worker execution instructions. `user_description` is the clean description shown in UI and hot-path context, while `taskPrompt` is the worker's standalone execution task.
- **Pattern model**: Patterns store trigger config, trigger filters, connector scope, notify condition, worker type, timezone, and activity state. Scheduled Patterns use top-level `triggerType: "schedule"` with canonical `schedule.kind` values (`once`, `interval`, `daily`, `weekly`, `monthly`) plus optional start boundaries; cron is internal-only for migrated/advanced schedules. Pattern create/edit tools do not accept timezone overrides or runtime-scope overrides; schedules use the user's effective timezone (`manual` or `browser`, falling back to `USER_TIMEZONE` while source is `server`). Keep docs and runtime behavior aligned when any of these fields change.
- **Pattern editing**: Existing Patterns should usually be updated in place so the same Pattern ID and `pattern_runs` history survive schedule/trigger changes. Delete/recreate should be reserved for true replacement or failed in-place edits.
- **Pattern management tools**: Pattern CRUD/discovery is exposed only to `pattern_management` workers through Finn JS workspace APIs such as `finn.patterns.list`, `finn.patterns.inspect`, `finn.patterns.create`, and `finn.patterns.edit`. `finn.patterns.list({})` is paginated and summary-only (ID, name, user description); use `finn.patterns.inspect({ id })` for full details before edits/deletes/replacements.
- **Pattern run history**: `pattern_runs` persists trigger payloads, skip reasons, connector tool scope, notify outcomes, and `surfacedAt`. Pattern workers receive compact recent-run context plus the read-only `finn.pattern.runs` / `finn.pattern.run` APIs for full history when needed.
- **My Day model**: My Day is a first-class Postgres daily summary/todo surface and works without a memory provider. Background refreshes require at least one connector with My Day enabled and run only at user-local times from `MY_DAY_REFRESH_TIMES` (default `0500,1100,1700`) or via manual refresh. Connector connection/config changes must not queue My Day LLM refreshes. Opening the web My Day sheet must not trigger an LLM refresh. Refreshes may update the summary, add high-confidence source-backed todos, and edit/archive stale refresh-created todos that have not been handed off; they must not complete todos or modify/archive user-created todos.
- **My Day todo lifecycle**: Open todos roll forward across days from their creation day. Completed todos remain visible only on the user-local day they were completed. Deleting a My Day todo archives it so it is hidden from the UI but retained as recent dedupe context. Refresh-created todos must be deduped against open, same-day completed, and recent archived todos. Handoff routes a user-context message into the hot path; Finn decides whether to ask for clarification, delegate work, or mark the todo done. A todo with `handoffAt` cannot be handed off again.
- **Connector automation defaults**: New connectors default My Day and Personal Intelligence off, except primary required mail connectors (`gmail`/`outlook` from `requiredComposioToolkits`) which stay enabled and cannot be disconnected from the web UI. Users opt in per non-primary connector from the web app. Enabling My Day affects the next scheduled My Day run; enabling Personal Intelligence can queue scoped ingestion for that connector only after Finn verifies a stable account identity.
- **Personal Intelligence**: Personal Intelligence is memory-provider-backed and connector-scoped. It runs at user-local times from `PERSONAL_INTELLIGENCE_REFRESH_TIMES` (default midnight) and fans out to one ingestion run per enabled connector/account scope. Composio-backed connectors must resolve a stable `accountScopeId` before PI can be enabled and must load only that connector's read-only Composio tools. Puter connectors use the fixed Puter PI account scope and must load only Puter toolsets that have the separate Personal Intelligence opt-in and talk live to the paired Mac app through `PuterBridge`; never batch-upload local iMessage/Notes records. Before starting a Puter PI run, verify the paired device matches the connector account and is actively connected. PI should recall before retaining for semantic/source dedupe, retain incrementally through the provider-neutral tool path, preserve source provenance and user connected-account perspective, inspect readable attachments on interesting emails/messages when connector tools expose attachment text, downloadable paths, or URLs, and capture durable cross-app project/workstream context when records show active responsibilities or decisions. It keeps a retained-source ledger plus connector-agnostic coverage checkpoints per user/toolkit/accountScope/source type so each connector's initial backfill stays bounded (default 30 days) and future runs continue from that account scope's prior coverage with overlap instead of rescanning old information. Disabling and re-enabling a connector must preserve prior checkpoint/source state when the stable account scope is unchanged, even if the provider connected-account id changes.
- **Pattern outcomes**: Pattern-triggered workers must finish with an explicit notify contract (`notify`, `summary`, optional `reason`, `data`). The scheduler enforces notify outcomes and notify conditions server-side before the hot path surfaces Pattern work.
- **Pattern setup confirmations**: Pattern creation outcomes that the hot path may surface should carry structured setup data copied from `create_pattern` (`type: pattern_setup`, `patternId`, `patternName`, `triggerType`, `nextRun`). Hot path confirmations must use persisted `nextRun` when present and must not invent one for toolkit/event-triggered Patterns.
- **Optional features**: Integrations toggle on when their API keys are present and selected. The web research provider is selected by `WEB_SEARCH_PROVIDER` (`auto`, `exa`, `parallel`, or `none`); Exa/Parallel web research, Fal, Composio, STT, and TTS capabilities should not be exposed to LLMs unless configured and actually wired.
- **Composio runtime**: Composio is worker-only. Its native toolset is loaded per worker run, scoped to the configured user, and omitted when no configured toolkits/tools are available. Web connector authorization syncs config but should not route connector lifecycle messages through the hot path or queue My Day refreshes. Composio sync must not mutate project-owned connectors such as Puter, which are paired and tracked outside Composio.
- **Puter runtime**: Puter is a paired local-Mac connector, not a Composio connector. The web app must show setup-only Puter UI until a Mac app is paired, and source toggles must stay hidden/blocked until then. Paired-but-closed Puter should show Offline, not Connected. Finn Puter persists only non-secret host/device setup state in its config file, stores the copied web session token in macOS Keychain, uses a random per-install device ID, hides to the menu bar on window close, and exits only through Quit Puter. Finn Puter keeps an outbound WebSocket session to the server while signed in, even when no local source is enabled; before the first socket-ready event it should attempt connection once and show a retry view on failure, then after the first successful connection it should automatically reconnect on later drops. Socket-token requests from the signed-in Mac app may re-pair a stale device id for that same user only, and token issuance plus queued commands must stay capped. Server-side LLM processes can use enabled Puter tools only while that active session exists and every local command must be gated by a run lease. Personal Intelligence is a per-source sub-option, separate from general Puter tool access. If a scheduled Puter PI run is missed because the Mac is offline, the next live socket connection should defer-run the most recent refresh slot only when that connector has not completed the current local PI window or the enabled Puter PI sources were not covered. Multiple LLM runs must share the same per-user/per-device session through command IDs and run leases, never through global state. Local Puter reads must clamp to the server-provided PI window, exclude Messages rows hidden from the user by archive/deleted/recoverable/spam/retracted state, preserve Apple `message.is_from_me` as `metadata.isFromMe`, and normalize those sent rows to `sender: "me"`/`metadata.localUser: true` without hardcoded user names or handles. Keep local subprocesses timeout-bounded and the Tauri CSP restrictive.
- **Project-local toolsets**: `packages/toolsets` is the home for Finn API/toolset definitions. Worker-facing families such as `files`, `web`, `creative`, `mcp`, `patterns`, `pattern`, and Puter toolsets are exposed through Secure Exec Finn JS workspace with generated searchable API docs, runtime requirements, and grant-filtered command surfaces. Hot-path file context uses the `files` APIs through Finn JS workspace with write-capable workspace access and document extraction suppressed. Keep toolsets project-wide, allowlisted, and gated by existing connector/runtime policy. Do not advertise or load a toolset unless the runtime is explicitly allowed to use it.
- **User runtimes**: Top-level ingress, routes, and Graphile jobs identify `{ tenantId, userId }` first, then call `UserRuntimeRegistry.ensure()` / accessors. User-scoped services must be resolved from `UserRuntimeServices` or `ProcessRuntimeServices`; do not derive user workspace roots, construct `FileStorage`, create memory recorders, or instantiate MCP/Finn JS workspace resources outside `packages/server/src/user-runtime.ts`.
- **Process runtime views**: Use `createProcessRuntimeServices()` to narrow a user runtime per process/run. Process views grant only the required runtime slots (`files`, `web`, `creative`, `artifacts`, `mcp`, `patterns`, `memory`, etc.), set file access (`read` or `write`), and must not expose a back-reference to the full user runtime. LLM processes may receive `workspace_search` and `workspace_execute`; `read` file access means `/workspace` is read-only through Finn files APIs, while `/artifacts` remains available for run-scoped outputs.
- **User workspaces**: Every user runtime gets an isolated root at `${WORKER_WORKSPACES_PATH}/${tenantId}/${userId}`. Finn-managed internal artifacts live under `.finn/`, while the LLM-visible workspace is mounted from `workspace/` with user-owned files under `workspace/files/` and reserved skills under `workspace/skills/`. Run-scoped temporary files that files tools need to inspect should use `/artifacts/...`; VM-local scratch stays under `/tmp`. Skills remain unavailable to Finn until explicitly enabled.
- **Finn JS workspace filesystem boundary**: Finn files APIs address the user-visible workspace at `/workspace` and run artifacts at `/artifacts`. Secure Exec-local `/tmp` is sandbox scratch only and is not a Finn files path. Read-only process access applies to `/workspace`; `/artifacts` is the run-scoped place for temporary outputs that should be scrubbed with the process. Do not rely on scratch directories for policy; contain access at the user workspace boundary and through Finn-owned runtime/tool grants.
- **Process file access**: File access is narrowed per process through runtime views: hot path/general workers/Pattern workers are write-capable, while `pattern_management`, My Day, and Personal Intelligence are read-only. Public file URLs include tenant and user path segments (`/files/:tenantId/:userId/:fileId`); do not add ownerless lookup or legacy `/files/:id` routes.
- **Worker tool-output artifacts**: Oversized worker tool results are spilled into run-scoped `/artifacts/...` outputs instead of being carried inline forever. Follow-up worker context may inspect those paths through `workspace_search` and `workspace_execute` with `finn.files.*` during the run, but artifact paths are transient and must be summarized before final outcomes.
- **Document extraction**: Document extraction is a local-only `finn.files.extract` Finn JS workspace capability for PDFs, DOCX, spreadsheets, HTML/text, and LibreOffice-convertible Office files. Use `finn.files.extract`, not legacy native worker document tools. PDF extraction prefers `@firecrawl/pdf-inspector`, falls back to local Poppler `pdftotext`, and can OCR scanned/flattened PDFs through local OCRmyPDF/Tesseract. It is exposed to eligible general/Pattern workers and read-only automation processes through runtime files access. Connector-provided URL reads must reject local/private/internal network targets, including redirect targets, before fetching.
- **Worker Finn JS workspace runtime**: Worker toolset execution must run through process-scoped Secure Exec Finn JS workspace. Model-facing worker instructions should refer to `workspace_search`, `workspace_execute`, the global `finn` object, `/workspace` user-visible paths, and `/artifacts` run artifacts. Secure Exec is not a shell: do not expose host child processes, package managers, long-running sessions, or shell-parity tools to LLM workers.
- **Web app shell**: `packages/web` is a Vite React app with a standalone-oriented PWA shell (`manifest.webmanifest` + `public/sw.js`). The session endpoint returns Finn's Spectrum line number when configured so the dashboard can deep-link into SMS.
- **Dual lockfiles**: `bun.lock` (primary) + `pnpm-lock.yaml` (legacy). Use Bun. Docker dependency installs must copy `patches/` before `bun install --frozen-lockfile` because Bun needs patched dependency files during install.

## ANTI-PATTERNS (THIS PROJECT)

**Hard rules from prompts/identity (enforced by LLM behavior):**
- **Never fabricate information** — workers and hot-path must not make up data
- **Never announce memory access** — no "searching memory" or revealing memory mechanics to user
- **Never execute irreversible actions without explicit user confirmation**
- **MUST delegate immediately** for scheduled/time-based tasks (hot-path is stateless for these)
- **Never promise without delegating** — if committing to a future task, must call delegate tool
- **Workers must NOT communicate with the user** — workers are invisible; only hot-path texts
- **Compactor must not fabricate** — preserve facts exactly; no meta-commentary about summarization
- **Lowercase always** in Finn's messages; never split replies into multiple messages
- **Reactions**: Use react tool only, never output reaction text; stick to exact tapback names

**Code-level:**
- No ESLint/Prettier configured — rely on TypeScript strict mode + Bun defaults
- Tests exist across core runtime packages; add focused regression coverage for behavior changes, especially capability-gated combinations.
- No CI/CD pipelines — deployment is Docker Compose only
- Container entrypoint auto-runs migrations (`bunx drizzle-kit migrate`) on every start

## COMMANDS

```bash
# All commands from repo root
bun install                    # Install deps (use Bun, not pnpm)
bun run dev                    # Dev server with hot reload
bun run start                  # Production server
bun run build                  # Bundle for production (bun build → dist/)
bun run check                  # Type-check all packages (tsc --noEmit)
bun run test                   # Run Bun test suite
bun run db:generate            # Generate Drizzle migrations
bun run db:migrate             # Run pending migrations
bun run db:push                # Push schema directly to DB
bun run db:studio              # Open Drizzle Studio GUI
docker compose up -d           # Start Finn + Postgres
docker compose up postgres -d  # Start only Postgres
docker compose --profile tunnel up -d  # Include Cloudflare tunnel
docker compose up -d --build finn      # Rebuild after code changes
docker compose restart finn            # Pick up identity/prompts changes
```

## NOTES

- **Workspace root is the repo root** — `.git` and `package.json` now both live at `FinnAI/`.
- **Prompts and identity are runtime code** — they're bind-mounted into Docker and loaded at startup. Changes require `docker compose restart finn` (no rebuild).
- **Models configurable per process** — `HOT_PATH_MODEL`, `WORKER_MODEL`, and `COMPACTOR_MODEL`. Format: `provider:model-name`; supported providers are `anthropic`, `openai`, `fireworks`, `deepseek`, and `openai-compatible`. OpenAI-compatible endpoints also require `DEFAULT_BASE_URL` or the process-specific `*_BASE_URL` override, including the `/v1` prefix. Defaults to Anthropic Claude Sonnet for hot-path/worker and OpenAI GPT-4o-mini for compactor.
- **Hot-path ingress envs** — the live config uses `HOT_PATH_INGRESS_USER_GROUPING_WINDOW_MS` and `HOT_PATH_INGRESS_MAX_COALESCE_MESSAGES`.
- **Compaction thresholds** — context management triggers compaction at 70% (warn), 80% (background), 90% (aggressive), 95% (emergency) of max context tokens (default 128K).
- **Daily handoff summaries** — chapter rollover handoffs should stay focused on live personal continuity, not durable profile facts or Pattern operations that already live elsewhere in runtime context.
- **MCP servers** — external tool servers stored in config/DB. `alwaysOn` servers auto-connect on startup. MCP manager falls back from streamable-http to SSE. Web-managed MCP auth secrets are stored per-user in server-only workspace files, not in shared MCP rows. GitHub's hosted remote MCP currently works via token header auth (`Authorization: Bearer <token>`), not Finn's generic MCP OAuth path.
- **Voice flow** — STT and TTS are independent capabilities. Deepgram enables inbound audio transcription; ElevenLabs enables explicit `send_message({ voice_message: true })` replies. Voice replies synthesize audio, convert it to `.caf` with ffmpeg/libopus in the user workspace temp directory, store it under the user's workspace, then send via Spectrum voice content.
- **Web app behavior** — the dashboard offers sheets for profile, My Day, connectors, and patterns, plus recent Pattern run history and a `Text Finn` SMS deep link to the user's stored Spectrum assigned line, falling back to `SPECTRUM_DEDICATED_LINE_PHONE` only for dedicated-line deployments. On authenticated dashboard load, the browser timezone is captured once with `timezoneSource: "browser"` only when the profile still uses `server`; never overwrite `browser` or `manual` automatically. Sheet state mirrors the URL with `replaceState` instead of accumulating browser history entries. Toasts use `react-hot-toast`; when sheets are open, the toast viewport is portaled into the active Silk sheet layer so it stays above the sheet. The initial app screen is `loading` until `/api/web/session` resolves to avoid flashing login for authenticated users.
- **Complexity hotspots** (files >300 lines): `server/src/index.ts` (367), `integrations/src/mcp-manager.ts` (412), `integrations/src/fal.ts` (349), `core/src/config.ts` (302).

## PULL REQUESTS

- Use the templates in `.github/PULL_REQUEST_TEMPLATE/` when preparing PR content. Pick the closest fit: `standard`, `feature`, `bugfix`, or `hotfix`.
- Follow `CONTRIBUTING.md` for contributor-facing setup, validation, and documentation expectations.
- PR descriptions should call out the Finn runtime surfaces affected, the highest-risk behavior change, exact validation commands run, and any rollout notes such as config changes, migrations, or prompt/identity restarts.
- Do not assume contributors or reviewers have access to Linear. Link related GitHub issues or discussions when they exist, but keep the PR usable without internal tooling.
- When prompts, identity, tools, capabilities, migrations, or runtime boundaries change, mention that explicitly in the PR summary so reviewers can reason about product and operational impact quickly.

---
> Source: [ambrosecltr/ProjectFinn](https://github.com/ambrosecltr/ProjectFinn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
