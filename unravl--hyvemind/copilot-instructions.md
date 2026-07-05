## hyvemind

> > **Read `PRODUCT.md` first.** It contains the product overview (what Hyvemind is, the three systems, the bee colony agents, key differentiators, current alpha state, brand, business model, roadmap, glossary). This file is the **technical/operational reference** — file paths, IPC surface, debug commands, investigation guides — and deliberately does **not** repeat product context.

@PRODUCT.md

# Hyvemind — Technical & Operational Reference

> **Read `PRODUCT.md` first.** It contains the product overview (what Hyvemind is, the three systems, the bee colony agents, key differentiators, current alpha state, brand, business model, roadmap, glossary). This file is the **technical/operational reference** — file paths, IPC surface, debug commands, investigation guides — and deliberately does **not** repeat product context.
>
> **Terminology**: Hyvemind has no "Chat" surface. Wherever this file says `chat.rs`, `chat-event`, `chat-sessions/`, etc., these are internal/historical names for the **Tasks-view conversation**. See `PRODUCT.md §3` for details.

## How to use this document

This file is the technical reference for AI coding agents and humans working on Hyvemind. The major sections you can jump to:

- **[Documentation Maintenance](#documentation-maintenance)** — the rule + pointer to the full trigger tables (`docs/doc-maintenance.md`). Consult it after any change touching IPC / events / env-vars / new files / exported types.
- **[Quick Reference](#quick-reference)** — common dev commands.
- **[Content Security Policy](#content-security-policy-csp)** — the renderer's CSP and why each directive is what it is.
- **[OpenCode runtime](#opencode-runtime)** — how the external OpenCode agent runtime is resolved and managed.
- **[Crash recovery](#crash-recovery)** — what's reconciled on the next launch and what's lost.
- **[Project Layout](#project-layout)** — Rust backend file tree.
- **[Frontend Architecture](#frontend-architecture)** — React 18 + Vite + Tailwind shell, screens, providers, IPC layer, event listeners.
- **[Key Types](#key-types)** — the types most code paths route through.
- **[Diagnosing Hyvemind entities](#diagnosing-hyvemind-entities)** — START HERE for any investigation: the one-call `hyvemind-diagnose` skill.
- **Investigation guides** — manual fallback recipes for [Tasks](#investigating-a-task), [Sessions](#investigating-a-session-id), [Hivemind Reviews](#investigating-a-hivemind-review), and [Swarms](#investigating-a-swarm--progress_logjsonl).
- **[Tauri Commands](#tauri-commands-ipc)** — IPC surface.
- **[Tauri Events](#tauri-events-backend--frontend)** — streaming event channels.
- **[Environment Variables](#environment-variables)** — credentials, debug flags, tunables.
- **[Debug Mode](#debug-mode-checking-logs)** — how to read the structured logs Hyvemind writes when `HYVEMIND_DEBUG=1`.

### Sibling documentation

Don't duplicate any of these in CLAUDE.md — link to them:

| Doc | What it covers |
|-----|----------------|
| `PRODUCT.md` | Product context: vision, three systems, bee-colony agents, brand, roadmap, glossary |
| `CONTRIBUTING.md` | Dev setup, PR checklist, how to add a provider |
| `CHANGELOG.md` | Release notes; `[Unreleased]` accumulates work since the last tag |
| `SECURITY.md` | Threat model, vulnerability reporting process |
| `CODE_OF_CONDUCT.md` | Contributor Covenant v2.1 |
| `app/src/A11Y.md` | Frontend accessibility conventions (aria-live, screen-reader rules) |
| `docs/README.md` | Index of all topical / deep-dive docs + the per-subsystem README map (read this to find a doc; this is the index the now-removed `AGENTS.md` used to provide) |
| `docs/architecture.md` | System component map + sequence-flow diagrams (Mermaid) + crash-recovery + storage layout |
| `docs/ipc-reference.md` | Per-command reference for every Tauri IPC handler (signature / returns / errors / delegation) |
| `docs/frontend-architecture.md` | React deep-dive: provider tree, `taskRuntime.tsx` sub-contexts, IPC wrapper, singleton event stores, recipes |
| `docs/bee-agents.md` | Per-agent deep-dive (Queen / Scout / Worker / Guard / Nurse + scout-review + stability-test pair) |
| `docs/providers.md` | LLM provider abstraction overview (the 4 production impls + mock, dispatch, cost, circuit-breaker integration) |
| `docs/extension-authoring.md` | How to author Rust provider extensions and frontend topbar widgets (+ the OpenCode `submit_*` echo-tool generation) |
| `docs/developer-cookbook.md` | Task-oriented recipes (add a command / tunable / screen / migration, point at a custom opencode binary, enable debug, etc.) |
| `docs/doc-maintenance.md` | Doc-authoring guide: which change requires which CLAUDE.md / PRODUCT.md / README section edit + the drift-audit commands |
| `docs/hivemind-custom-prompts.md` | Ready-to-paste Hivemind reviewer prompts |
| `app/src-tauri/src/core/README.md` | Swarm execution engine deep-dive |
| `app/src-tauri/src/hivemind/README.md` | Multi-model review engine deep-dive |
| `app/src-tauri/src/state/README.md` | Persistence, AppState, secret store, atomic writes |
| `app/src-tauri/src/extensions/README.md` | Provider usage/balance extension framework |
| `app/src-tauri/src/nurse/README.md` | Nurse push-mode engine — bus, detectors, three-tier dispatch pipeline, always-on observability |
| `app/src-tauri/src/providers/README.md` | LLM provider abstraction deep-dive (trait surface, the 5 impls, add-a-provider walkthrough) |
| `app/src-tauri/src/domain/README.md` | Domain type glossary (`SwarmState`, `Feature`, `Milestone`, `ModelSettings`, transitions) |
| `app/src-tauri/src/harness/README.md` | Agent-runtime harness deep-dive (`AgentHarness` / `HarnessSession` traits, host-owned `SessionRuntime` + single-owner `SessionManager`, the `HarnessCapabilities` model, the OpenCode backend, adding a backend) |

## Documentation Maintenance

> **The rule**: when you change something the docs describe, update the doc in the same commit.

The full trigger tables (which change requires which `CLAUDE.md` / `PRODUCT.md` / `README.md` section edit), the recurring drift-audit commands, and the "what NOT to put in CLAUDE.md" list live in **[`docs/doc-maintenance.md`](docs/doc-maintenance.md)** — they're doc-authoring guidance, not coding context, so they don't earn their tokens on every session. Section names there (e.g. "§Tauri Commands (IPC)") refer back to this file.

**Consult `docs/doc-maintenance.md` whenever your change:**

- adds/removes a `#[tauri::command]`, a Tauri `emit("…", …)`, or a `HYVEMIND_*` env var / `tunables.rs` default
- adds or deletes a file under `src-tauri/src/{commands,core,harness,hivemind,nurse,state,extensions,providers,domain}/` or `prompts/`
- changes an exported `pub` type, a harness trait method / `HarnessCapabilities` flag, or the CSP in `tauri.conf.json`
- ships a feature, adds a screen, or renames a user-facing concept

Skipping it is how the docs drift.

## Quick Reference

```bash
# Rust backend
cd app/src-tauri
cargo check                    # type-check
cargo test                     # full backend test suite (~1,040 #[test] annotations)
cargo build                    # release build
cargo fmt --check              # CI-mirror format check
cargo clippy -- -D warnings    # CI-mirror lint

# Frontend (React + Vite)
cd app
npm test                       # Vitest run-once (~75 test files)
npm run test:watch             # Vitest in watch mode
npx tsc --noEmit               # CI-mirror type check
npm run dev                    # Vite dev server (port 1430)

# Full Tauri shell (Rust + Vite; requires a user-installed `opencode` binary for agent features)
cd app
npm run tauri:dev              # dev mode with HYVEMIND_DEBUG=1
npm run tauri:build            # production bundle
```

## Content Security Policy (CSP)

The Tauri WebView runs under a strict CSP defined in `app/src-tauri/tauri.conf.json` under `app.security.csp`. The policy is:

```
default-src 'self';
script-src 'self';
style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
img-src 'self' data: blob: asset: http://asset.localhost https://asset.localhost;
connect-src 'self' ipc: http://ipc.localhost https://ipc.localhost http://localhost:* ws://localhost:*;
font-src 'self' data: https://fonts.gstatic.com;
object-src 'none';
base-uri 'self';
frame-ancestors 'none';
form-action 'self'
```

**Why each directive looks like this**

| Directive | Allowed | Reason |
|-----------|---------|--------|
| `default-src` | `'self'` | Deny-by-default fallback for any directive not listed below. |
| `script-src` | `'self'` only | No inline JS, no `eval`. The Vite production build emits bundled scripts only, and the codebase has no `dangerouslySetInnerHTML`/`eval`/`new Function`. **Do not add `'unsafe-inline'` or `'unsafe-eval'` here** — that's the single biggest XSS lever. |
| `style-src` | `'self'`, `'unsafe-inline'`, `https://fonts.googleapis.com` | Tailwind + Vite inject runtime `<style>` blocks (this is unavoidable without nonces, which Tauri doesn't wire up). `fonts.googleapis.com` hosts the `@import`-style CSS for Inter and JetBrains Mono loaded from `app/index.html`. |
| `img-src` | `'self'`, `data:`, `blob:`, `asset:`, `http(s)://asset.localhost` | `data:` for inline SVG backgrounds in `src/index.css`. `blob:` for `URL.createObjectURL` image-attachment previews in Tasks/Chat. `asset:` and `asset.localhost` are Tauri 2's asset protocol, in case future features serve local files to the WebView. |
| `connect-src` | `'self'`, `ipc:`, `http(s)://ipc.localhost`, `http://localhost:*`, `ws://localhost:*` | Tauri 2 IPC bridge uses `ipc:`/`ipc.localhost` (platform-dependent). `http://localhost:*` covers the Vite dev server (port 1430). `ws://localhost:*` is a defensive allow for any future Vite HMR (HMR is currently disabled in `vite.config.ts`). The renderer makes no direct external HTTP calls — all LLM provider traffic goes through Rust via Tauri IPC. |
| `font-src` | `'self'`, `data:`, `https://fonts.gstatic.com` | `fonts.gstatic.com` serves the actual font binaries Google Fonts hands out. `data:` is permitted for any tooling that inlines a fallback glyph. |
| `object-src` | `'none'` | No `<object>`/`<embed>`/Flash. |
| `base-uri` | `'self'` | Prevents `<base href="...">` injection redirecting relative URLs to an attacker host. |
| `frame-ancestors` | `'none'` | The Tauri window can't be embedded in another frame. |
| `form-action` | `'self'` | Defensive; no real `<form>` submits happen, but locks down `<form action="...">` injection. |

**Things to watch when modifying the frontend**

- Adding a new external CDN (image, font, analytics, etc.) requires updating the corresponding directive. Prefer self-hosting.
- Sentry events ship via the `tauri-plugin-sentry` IPC bridge, not a direct browser call to `*.ingest.sentry.io`. If that ever changes to direct ingest, add the Sentry host to `connect-src`.
- If you switch `react-markdown` to render raw HTML (`rehype-raw` or similar) you reintroduce XSS risk; the current strict policy assumes markdown is rendered through React components only.
- All LLM provider HTTP traffic is made by Rust (`reqwest`), not the WebView. Keep it that way — moving any provider call into the frontend would require opening `connect-src` to that vendor.

## OpenCode runtime

OpenCode ([github.com/sst/opencode](https://github.com/sst/opencode)) is the agent runtime behind every agentic session (Tasks, Hivemind context/merge, all bee roles). It is **user-installed, never bundled** — there is no build pipeline, no `externalBin`, and nothing under `binaries/`. The recommended install is `curl -fsSL https://opencode.ai/install | bash` (the Settings screen shows this hint when the binary is missing).

- **Binary resolution** (`harness/opencode/mod.rs::resolve_binary`): the `HYVEMIND_OPENCODE_BIN` env override wins; otherwise `opencode` on `PATH`, then `/opt/homebrew/bin/opencode`, then `/usr/local/bin/opencode`.
- **Shared server**: one `opencode serve` subprocess on loopback (`127.0.0.1`, OS-assigned port) hosts **all** sessions over HTTP + SSE. `OpenCodeHarness` lazily spawns (and respawns) it; an exit-monitor task broadcasts a fail-fast error to every session channel if the server dies. There is no per-session OS process and no process-pool semaphore.
- **Session resume**: a persisted id map at `~/.hyvemind/opencode/sessions.json` maps Hyvemind session ids → OpenCode server-side session ids. Spawns with `SessionOptions.resume = true` reattach when the server-side session still exists, falling back to a fresh create.
- **Availability gate**: `get_opencode_status` (Settings bucket) returns `{ installed, path, version }`; the frontend's `RuntimeGate` locks the agent-dependent tabs when `installed` is false. `AppState` constructs `OpenCodeHarness` unconditionally and warns if the binary is missing.
- **Custom provider auto-registration**: custom **OpenAI-compatible** providers from Hyvemind Settings (neokens, crof, …) are injected into the server as inline JSON config via the `OPENCODE_CONFIG_CONTENT` env var (OpenCode merges it over the user's global/project config). Generation is `Config::opencode_config_content` (`state/config.rs`); the env assembly chokepoint is `AppState::build_harness_env` / `harness_env` (`state/app_state.rs`) — **every `update_env_vars` call site must go through it** or the generated config silently drops on the next spawn. Blocks reference API keys as `{env:{PROVIDER}_API_KEY}` (never inline), and skip `OPENCODE_NATIVE_PROVIDERS` (anthropic/openai/openrouter/deepseek/mistral/groq/google) plus all non-OpenAI-compatible provider types. Each block's `models` map comes from the persisted `/models` cache at `~/.hyvemind/opencode/provider-models.json` (`state/provider_models.rs`), populated by `test_provider_models`, `add_provider`/`save_api_key` hooks, and a best-effort startup refresh task — unioned with any model ids the config references (`default_model`, `model_context_window_overrides`). Takes effect on the **next server spawn**, same as key updates.
- **Capabilities**: OpenCode declares `STRUCTURED_TOOL_ARGS | SESSION_STATS | RESUMABLE | MULTIMODAL_IMAGES` — no `PROCESS_LIVENESS`, no `FORCEFUL_KILL`, no `RETRY_EVENTS`. `MULTIMODAL_IMAGES` is a transport capability for image file parts; provider/model vision support is still model-dependent and unsupported models should fail through OpenCode/provider errors. Nurse degrades accordingly (cooperative kill path, no retry-exhaustion signal).
- **Hyvemind custom agents**: `harness/opencode/agents.rs` generates three agent definitions at server spawn (`agents/hyvemind-{coder,readonly,guard}.md`, written to the server cwd AND the OpenCode global config dir, same placement rules as the `submit_*` echo tools). Scripted sessions (any session with a Hyvemind system prompt) are routed onto them per tool-set shape via the per-message `agent` field, so the agent's compact `prompt` REPLACES OpenCode's 5–15 KB provider base prompt (the per-message `system` role prompt appends after it) and the agent `permission` block mechanically enforces read-only/guard tool policy. Plain Tasks conversations (no Hyvemind system prompt) stay on OpenCode's default `build` agent. Gated on `OpenCodeServer::agents_available` — an unknown agent name errors the turn server-side.

## Crash recovery

After an ungraceful exit (host crash, force-quit, OOM, power loss), Hyvemind reconciles three classes of orphaned state on the next launch. Anything not in this list is **lost**.

| What | Where it's reconciled | What gets recovered | What's lost |
|------|----------------------|--------------------|-------------|
| Tasks-view conversations | No startup pass needed — OpenCode persists sessions server-side, and the Hyvemind→OpenCode id map at `~/.hyvemind/opencode/sessions.json` survives the crash | The next `send_message` against a known `session_id` spawns with `resume = true`; `OpenCodeHarness` reattaches to the server-side session via the id map (falling back to a fresh create when the server-side session no longer exists). Frontend message history is independent — it lives in `~/.hyvemind/task-messages/`. | Any tokens that were mid-stream when the host crashed. The in-memory `SessionRuntime` transcript (so `get_chat_history` for the old runtime returns empty until new turns arrive). |
| Orphan "Implementing" swarms | `reconcile_orphaned_swarms` + `migrate_legacy_reconciled_failures` early in `AppState::new` | Each on-disk swarm still tagged `Implementing` is rewritten to `Interrupted` with an explanatory error message; the user can click Resume to continue from where the Queen left off. `Paused` swarms are intentionally untouched so user-issued pauses survive a restart. | In-flight Worker/Guard sessions (their OpenCode server-side state is simply abandoned; Resume starts fresh agent runs). |
| Orphan Hivemind reviews / merges | `hivemind_store.sweep_interrupted_merges()` and `.sweep_interrupted_jobs()` in `AppState::new`; rows are then drained via `take_pending_interrupted_emits()` and emitted to the frontend as `merge_interrupted` / `review_interrupted` Tauri events after `app.manage()` | Any `merge_run` left `running` or any job left `pending` / `running` / `round_*` is flipped to `interrupted` so the UI can offer recovery instead of showing a phantom spinner. | Partial round outputs that hadn't been persisted to SQLite at crash time. |

Startup replay of on-disk transcripts is gone — JSONL transcripts are no longer written or read. Chat continuity is OpenCode's server-side session store plus the `sessions.json` id map.

**`run_id` semantics** — the Queen mints one `run_id` per `run_feature_full` invocation (`core/queen.rs`). Scout, Worker, inline Guard, and validator Guard for the same attempt all share it. Format: `run-{feature_id}-{fix_attempt_count}-{uuid_simple}`. This is what lets `progress_log.jsonl` distinguish retries on the same feature, and what routes per-agent debug files (`debug/swarms/{swarm_id}/{agent}-{feature_id}-{run_id}.jsonl`). A feature with `fix_attempt_count == 3` will have 4 distinct `run_id`s in the log (original + 3 fixes).

**`Paused` is intentionally NOT rewritten** by the reconciler — a user-issued pause survives a restart. Only `Implementing` (and `Failed`-with-in-flight-features) flips to `Interrupted`.

**Process-enumeration-style reaping** (`ps`/`sysinfo` scan for an orphan `opencode serve` whose original parent PID is gone) is intentionally **not** implemented. The shared server child is spawned with `kill_on_drop(true)` and the OS's parent-death signalling on macOS/Linux handles the rest. Scanning the system process table to claim "ours" by argv heuristics is too invasive and racy to be reliable — and OpenCode's own server-side session store (which the id map points back into) is what matters for resuming a conversation, not the dead process.

## Project Layout

This is the Rust backend tree. For the frontend, see [Frontend Architecture](#frontend-architecture).

```
app/
  src-tauri/
    src/
      main.rs                    # Entry point — calls lib::run()
      lib.rs                     # Tauri builder, invoke_handler!, setup hooks, RunEvent::Exit teardown
      sentry_setup.rs            # Sentry client init (opt-in via config.crash_reporting)
      tunables.rs                # Centralised HYVEMIND_* env-var knobs (see §Tunables)
      commands/                  # Tauri IPC command handlers — thin delegators only, no business logic
        mod.rs                   # Module declarations
        chat.rs                  # send_message, stop_chat, get_chat_history,
                                 #   list_chat_sessions, delete_chat_session, is_session_busy,
                                 #   get_session_last_assistant_text
        tasks.rs                 # save/load/delete_task_messages, get_task_state, auto_commit_task,
                                 #   list_project_files
        hivemind.rs              # start_review, get_review_status, list_reviews, create/update/delete_hivemind,
                                 #   get_review_step_outputs, cancel_review, delete_review, save/list_round_verdicts,
                                 #   read_merge_output, register_context_session, get_orchestrator_usage,
                                 #   get_resumable_review_for_task, get_review_plan, get_review_state,
                                 #   log_review_event (+ debug-only clear_response_cache)
        swarms.rs                # create_swarm, update_swarm, start/pause/resume/stop_swarm,
                                 #   get/list/delete_swarm, get_swarm_progress/features/milestones/usage,
                                 #   check_swarm_readiness
        settings.rs              # get_settings, get_system_prompts, list/save/delete_custom_prompt,
                                 #   set_runtime_settings, set_default_model/hivemind/project_path,
                                 #   request_working_dir_approval, save/delete_api_key, get_providers,
                                 #   add_provider, list/add/update/delete_mcp_server,
                                 #   set_mcp_server_enabled, save_mcp_server_token,
                                 #   test_provider_models/chat, refresh_models,
                                 #   get_opencode_status, open_opencode_terminal, check_subscription_auth,
                                 #   set_auto_commit_tasks/conventional, set_task_completion_sound,
                                 #   set_crash_reporting, set_chat_check_in_secs,
                                 #   set_extension_poll_interval_secs, set_daily_budget
        dashboard.rs             # get_dashboard_stats, get_model_usage, get_provider_usage,
                                 #   get_cost_summary, get_recent_activity
        extensions.rs            # list_extensions, get_usage_snapshots, refresh_usage_snapshot,
                                 #   update_extension_settings
        nurse.rs                 # get_nurse_status, set_nurse_config, check_chat_session
        sessions.rs              # list_active_sessions, kill_session, reconcile_active_sessions,
                                 #   session_pool_stats, get_session_stats
        tests.rs                 # run_stability_test, cancel_test_run, get_active_test_run,
                                 #   list_test_runs, get_test_run, get/set_stability_test_config
        util.rs                  # path normalisation + shared helpers (no commands)
      core/                      # Swarm execution engine (see app/src-tauri/src/core/README.md)
        queen.rs                 # Orchestrator — decomposes goal, runs the integrated swarm loop
        scout.rs                 # Per-feature planner (read-only); produces implementation plan + risks
        scout_review.rs          # Optional Hivemind review of the Scout's plan before Worker handoff
        worker.rs                # Implementer; takes plan, writes code, calls the `submit_handoff` tool
        handoff.rs               # WorkerHandoff capture — reads `submit_handoff` tool args directly off the
                                 #   harness ToolExecutionStart event (STRUCTURED_TOOL_ARGS capture).
                                 #   No delimited-marker parser, no last-JSON-block fallback —
                                 #   missing tool call surfaces as `HandoffParseFailed` (Queen downcasts to a
                                 #   Nurse intervention bubble) and the feature fails.
        guard.rs                 # Milestone validator; synthesises fix-features (cap 3 attempts)
        scheduler.rs             # Topological sort, cycle detection, next_ready_batch() (bounded parallelism)
        swarm_context.rs         # Per-run context: cwd, git state, registries, model settings
        services.rs              # services.yaml parser — per-swarm command + ambient-service registry
        budget.rs                # Per-swarm + daily cost tracking; emits budget_exceeded
        readiness.rs             # Pre-flight checks (cargo crates, npm packages, system binaries)
        validation.rs            # Milestone-assertion validation helpers
        idempotency.rs           # Idempotency key generation/tracking for swarm ops
        recovery.rs              # Crash-recovery constants & helpers (e.g. INTERRUPTED_BY_RESTART_MSG)
        stability_test.rs        # Stability-test types + active-run tracking
        stability_test/
          runner.rs              # Test runner — spawns worker+guard, emits test-progress
      harness/                   # Agent-runtime abstraction + the OpenCode backend (see harness/README.md)
        mod.rs                   # AgentHarness + HarnessSession traits, BackendSignal, LoopState,
                                 #   preview() helper, canonical HYVEMIND_EXTENSION_TOOLS name list
        capabilities.rs          # HarnessCapabilities (Copy u32-newtype, 7 flags: PROCESS_LIVENESS,
                                 #   FORCEFUL_KILL, STRUCTURED_TOOL_ARGS, SESSION_STATS, RETRY_EVENTS,
                                 #   RESUMABLE, MULTIMODAL_IMAGES) + all() / all_except_forceful_kill /
                                 #   empty() / names()
        error.rs                 # HarnessError — the single error currency on the trait surface
                                 #   (TransportClosed / ProcessCrashed / Timeout / Unavailable /
                                 #   SessionNotFound / SessionExists / NotSupported / Io / Json / Backend)
        events.rs                # HarnessEvent enum (text/thinking deltas, tool start/update/end,
                                 #   lifecycle) + HarnessSessionStats + ToolTerminalStatus
        options.rs               # SessionOptions (model, thinking_level, tool_set, system_prompt, resume)
                                 #   + for_model/for_scout/for_worker/for_guard constructors +
                                 #   ThinkingLevel / ToolSet / HarnessImage / SessionOwner
        session_runtime.rs       # Host-owned SessionRuntime (Nurse-critical atomics + transcript + owner +
                                 #   bus fan-out + cancellation) wrapping the backend Arc<dyn HarnessSession>;
                                 #   the event-collection loops live here
        session_manager.rs       # SessionManager — the SOLE strong owner of every Arc<SessionRuntime>;
                                 #   spawn (register-then-publish) / close (remove-then-kill) / eviction
                                 #   hooks; SessionStatSnapshot
        eviction.rs              # Maintenance loop over the SessionManager registry: idle eviction
                                 #   (10-min TTL), needs_respawn sweep (SESSION_STATS-gated),
                                 #   auto_commit_locks sweep, session_pool_stats observability log;
                                 #   emits `session-evicted`
        chunk_sink.rs            # ChunkSink + forward_chunk_to_sink streaming-chunk helper
        context_window.rs        # Model context-window table + config override lookup
        mock_harness.rs          # MockBackend / MockAgentHarness / MockCommand (test/test-mocks gated) —
                                 #   the only test mock; reduced-cap variants exercise the cooperative
                                 #   kill-verify + capability-gating paths
        opencode/                # The OpenCode backend — shared `opencode serve` process, reduced
                                 #   capabilities (STRUCTURED_TOOL_ARGS | SESSION_STATS | RESUMABLE | MULTIMODAL_IMAGES)
          mod.rs                 # OpenCodeHarness spawn factory; lazy get-or-RESPAWN server; resume gating
                                 #   (options.resume) + persisted id map (<server_cwd>/sessions.json);
                                 #   HYVEMIND_OPENCODE_BIN / PATH / homebrew binary resolution;
                                 #   per-session Hyvemind-agent selection (agents::agent_for)
          agents.rs              # Hyvemind custom-agent markdown generation (hyvemind-coder /
                                 #   hyvemind-readonly / hyvemind-guard) into server cwd + the OpenCode
                                 #   global config dir; agent prompt REPLACES OpenCode's provider base
                                 #   prompt and the permission block mechanically enforces the tool policy
          server.rs              # Shared `opencode serve` child + HTTP/SSE client; per-directory SSE demux;
                                 #   exit-monitor task (fail-fast Error broadcast on crash); session_exists/
                                 #   attach_session resume plumbing; per-session channels + cached stats;
                                 #   spawn_tool_verification (GET /experimental/tool/ids)
          session.rs             # OpenCodeBackend (HarnessSession impl); cooperative force_kill RETAINS the
                                 #   server-side session; stale-AgentEnd classifier guard (saw_turn_activity);
                                 #   two-tier recv timeout (short streaming-silence window via tools_in_flight,
                                 #   long inactivity window while a tool runs) + best-effort abort on timeout
          event_map.rs           # Stateful SseMapper: OpenCode SSE → HarnessEvent — partID→kind tracking so
                                 #   reasoning deltas map to ThinkingDelta; idle latch dedupes the double
                                 #   session.status{idle}/session.idle AgentEnd
          tools.rs               # Tool allowlist translation (find→glob, ls→list) + submit_* echo-tool file
                                 #   generation into server cwd AND the OpenCode global config dir
      hivemind/                  # Multi-model review engine (see app/src-tauri/src/hivemind/README.md)
        mod.rs                   # Module root
        engine.rs                # ReviewEngine — JoinSet concurrency, per-round timeout, cross-review
        circuit_breaker.rs       # 3-state breaker (Closed/Open/HalfOpen) with probe_in_flight gating
        cache.rs                 # moka-based ResponseCache (lock-free, TTL- and size-bounded)
        backoff.rs               # Exponential backoff: min(60s, 5s*2^attempt + rand(0,2s))
        store.rs                 # SQLite store for review jobs/steps (sqlx, WAL mode)
        review_log.rs            # ReviewLogger — high-level event JSONL under ~/.hyvemind/reviews/
        review_schema.rs         # Review domain types (ReviewConfig, RoundConfig, Stance, etc.)
        merge_capture.rs         # Per-merge-run text capture under reviews/{id}/merge-r*.txt
        output_capture.rs        # Per-model-call output capture under reviews/{id}/output-*.txt
        events.rs                # ReviewEvent enum for progress emit
        phase.rs                 # Phase derivation helpers for progress UI
        verdicts.rs              # Round-verdict / decision types
        error.rs                 # Review error types
      nurse/                     # Nurse push-mode engine + dispatcher (see app/src-tauri/src/nurse/README.md)
                                 # The sole nurse dispatcher. Writes always-on observability to
                                 # ~/.hyvemind/debug/nurse/ regardless of HYVEMIND_DEBUG.
        mod.rs                   # Module root
        bus.rs                   # NurseBus — tokio::sync::broadcast<Arc<NurseBusEvent>>
        engine.rs                # NurseEngine — subscribes to bus, runs detectors, hands signals to Dispatcher
        dispatcher.rs            # Three-tier pipeline (Tier 1 deterministic / Tier 2 playbook / Tier 3 LLM)
                                 #   + InFlightGuard + tier1_lookup + EventSeq + Watchdog fast-paths
        health.rs                # SessionHealth, Signal, Severity, Tier, EscalationState
        detector.rs              # Detector trait, DetectorContext, DetectorRegistry, TickKind (Fast/Slow)
        detectors/
          mod.rs
          stall.rs               # Time-based + post-prompt silence stall detection
          reasoning_loop.rs      # Siphash exact / compression / minhash paraphrase loop detection
          tool_failure.rs        # Repeated tool failure clustering
          process_health.rs      # Backend liveness (PROCESS_LIVENESS-gated) + stderr crash patterns
          provider_health.rs     # Circuit-breaker / missing-provider signals
          context_saturation.rs  # Context-window % threshold (SESSION_STATS-gated)
          retry_exhaustion.rs    # Auto-retry-end clustering in a sliding window (RETRY_EVENTS-gated)
        classifier.rs            # LlmClassifier — Tier 3 wrapper around ProviderRegistry
        batch_review.rs          # BatchReviewer — ONE batched LLM call per tick across all active
                                 #   sessions (renders each session's recent transcript, parses
                                 #   per-session decisions, dispatches via dispatch_batch_decision)
        intervention.rs          # Synthesized dispatch + kill-verification constants
        intervention_writer.rs   # Bounded mpsc → in-memory ring (legacy IPC compat)
        playbook.rs              # Tier 2 templated steer table
        storm_guard.rs           # Per-(session, kind) sliding-window guard
        budget.rs                # Per-detector + age-decay intervention budget
        config.rs                # NurseConfig + NurseMode + NurseProfile + ProfileConfig + BudgetConfig
        schema.rs                # Provider-native tools/tool_choice JSON for the Tier 3 classifier
        prompt.rs                # include_str!("../../prompts/nurse_system.md") accessor
        snapshot.rs              # Wire DTOs (NurseDecision, NurseEvent, NurseLifecyclePayload, NurseStatusSnapshot,
                                 #   SessionOwnerDto, TunableDef, NurseDispatchTier, ...) consumed by the frontend.
                                 #   MonitoredSessionSnapshot carries harness_id + capabilities flag names
        synthesized.rs           # SynthesizedKind, InterventionOwner, describe_synthesized
        observability/           # Always-on JSONL surfaces under ~/.hyvemind/debug/nurse/
          mod.rs                 # ObservabilityHandles, prune_on_startup
          writer.rs              # JsonlWriter — bounded mpsc, non-blocking
          decision_log.rs        # DecisionLogger — daily-rotated decisions.jsonl.YYYY-MM-DD
          capture.rs             # ClassifierCapture — per-decision prompt/response files (1 MiB cap)
          signal_stream.rs       # SignalStream — per-session JSONL with 4 MiB rotation
          bus_telemetry.rs       # BusTelemetry — bus.jsonl.YYYY-MM-DD (lag, owner_changed, capacity_pressure)
      extensions/                # Provider usage/balance extension framework (see extensions/README.md)
        mod.rs                   # Module root, builtin registration
        traits.rs                # ProviderExtension / UsageProvider / BillingProvider traits
        types.rs                 # ExtensionManifest, UsageSnapshot, UsageMetric, ExtensionError
        registry.rs              # Extension registry + per-provider matching
        context.rs               # ExtensionContext (app handle, config access, http client, api_key lookup)
        poller.rs                # Per-extension polling loop (emits usage-snapshot-updated)
        tests.rs                 # Integration tests for registry, dedup, capabilities
        builtins/                # Built-in provider extensions
          mod.rs
          anthropic_usage.rs     # Anthropic usage/credit polling
          openrouter_credits.rs  # OpenRouter credit polling
          deepseek_balance.rs    # DeepSeek balance polling
          glm_coding_plan_usage.rs # GLM/Z.ai Coding Plan key/status + optional quota polling
          crof_usage.rs          # CROF usage polling
          claude_sub_usage.rs    # Claude subscription usage polling
          chatgpt_sub_usage.rs   # ChatGPT subscription usage polling
          neuralwatt_usage.rs    # Neuralwatt usage polling
          neokens_balance.rs     # neokens balance polling
      providers/                 # LLM backend dispatch (see app/src-tauri/src/providers/README.md)
        mod.rs                   # All provider impls (Anthropic, AnthropicOAuth, OpenAICompatible,
                                 #   OpenRouter, Mock) + ProviderRegistry + cost lookup
        provider_trait.rs        # Object-safe Provider trait + StreamingProvider extension trait
      domain/                    # Cross-subsystem type definitions (see app/src-tauri/src/domain/README.md)
        mod.rs                   # Re-exports
        swarm.rs                 # SwarmState, SwarmStatus, Feature, FeatureStatus, ModelSettings, Milestone
      state/                     # Persistence + app state (see app/src-tauri/src/state/README.md)
        mod.rs                   # Module root
        activity_log.rs          # Per-swarm append-only swarm-activity transcript; schema-versioned JSONL + paginated reader
        app_state.rs             # AppState — Tauri-managed root holding every subsystem Arc
        config.rs                # JSON config + env-var overrides; hot-reload-safe via clone-and-drop
        secret_store.rs          # OS keyring + AES-256-GCM-encrypted `.credentials` cache
        store.rs                 # SwarmStore — per-swarm directory layout, atomic writes (tempfile+rename)
        progress.rs              # Append-only JSONL progress log; crash-safe replay target (ProgressReader)
        provider_models.rs       # Persisted per-provider /models cache (opencode/provider-models.json)
                                 #   backing the generated OPENCODE_CONFIG_CONTENT provider blocks
        swarm_registry.rs        # RunningSwarm registry with CancellationToken + pause support
        usage_store.rs           # Aggregated usage/cost for Dashboard + topbar
        log_redact.rs            # API-key scrubbing writer wrapped around debug log files
        log_routing.rs           # PerIdRoutingLayer — routes tracing events to per-ID files on disk
        ipc_error.rs             # IpcError envelope returned by every command (mirrored in frontend)
        channel_drop.rs          # Channel-drop tracking utilities
        sync.rs                  # Misc sync primitives
      util/                      # Cross-cutting utilities
        mod.rs
        supervise.rs             # super_watchdog — respawns a tokio task once if the outer watchdog panics
    prompts/                     # Agent system prompts (most compiled in via include_str!)
      scout_system.md            # loaded by core/scout.rs
      worker_system.md           # loaded by core/worker.rs
      guard_system.md            # loaded by core/guard.rs
      context_system.md          # loaded by core/scout_review.rs (Hivemind context-gather agent)
      merge_system.md            # loaded by hivemind/engine.rs (default merge-orchestrator prompt)
      nurse_system.md            # loaded by nurse/prompt.rs (Tier 3 classifier system prompt)
      queen_system.md            # on disk but NOT include_str'd — Queen prompt is constructed dynamically
      queen_planning.md          # on disk but NOT include_str'd — planning sub-prompt
      plan_system.md             # on disk but NOT include_str'd — shared plan template
      stability_test_task.md     # loaded by core/stability_test/runner.rs
      stability_test_verifier.md # loaded by core/stability_test/runner.rs
    migrations/
      0001_hivemind.sql + subsequent migrations  # SQLite schema: jobs, job_steps, schema_migrations (0001) + later incremental migrations (usage log, hiveminds, orchestrator, round verdicts, indexes, …)
    Cargo.toml
    tauri.conf.json
  src/                           # Frontend (see §Frontend Architecture)
```

There is no bundled agent runtime — OpenCode is resolved from the user's machine at startup (see §OpenCode runtime). The `submit_*` echo tools OpenCode needs are generated at runtime by `harness/opencode/tools.rs` into `~/.hyvemind/opencode/.opencode/tool/` and the OpenCode global config dir.

## Key Types

| Type | Location | Purpose |
|------|----------|---------|
| `AppState` | `state/app_state.rs` | Tauri-managed root state; holds every subsystem `Arc` |
| `Config` | `state/config.rs` | JSON config loaded from `~/.hyvemind/config.json` with `HYVEMIND_*` env overrides. There is no harness-selection key — OpenCode is the only agent runtime. Notable keys: `model_context_window_overrides: HashMap<String, u64>` (per-model context-window override consumed by `harness/context_window.rs`), `mcp_servers` (CRUD'd from Settings but **not yet wired into OpenCode** — known limitation), `provider_env_vars()` folds provider keys into the OpenCode server environment, and `opencode_config_content(model_cache)` renders the inline OpenCode config (`OPENCODE_CONFIG_CONTENT`) that auto-registers custom OpenAI-compatible providers (see §OpenCode runtime). |
| `ProviderModelCache` / `CachedModel` | `state/provider_models.rs` | Persisted per-provider `/models` cache at `~/.hyvemind/opencode/provider-models.json` (atomic write, corrupt-tolerant load). Feeds the `models` map of each generated OpenCode provider block. Updated by `test_provider_models`, the `add_provider`/`save_api_key` hooks, and the startup `spawn_provider_model_refresh` task; every update recomputes the harness env via `AppState::build_harness_env`. |
| `SecretStore` | `state/secret_store.rs` | OS-keyring wrapper + AES-256-GCM-encrypted `.credentials` cache |
| `SwarmStore` | `state/store.rs` | Per-swarm directory layout, atomic writes via `tempfile + rename` |
| `ProgressLogger` / `ProgressReader` | `state/progress.rs` | Append-only JSONL progress log + crash-safe replay |
| `ActivityWriter` / `ActivityReader` | `state/activity_log.rs` | Per-swarm append-only `swarm-activity` transcript with monotonic `seq` per event; paginated reader powers `get_swarm_activity_log` history replay |
| `RunningSwarm` / `SwarmRegistry` | `state/swarm_registry.rs` | In-memory registry with `CancellationToken` + pause support |
| `UsageStore` | `state/usage_store.rs` | Aggregated usage/cost feeding Dashboard + topbar |
| `PerIdRoutingLayer` | `state/log_routing.rs` | Tracing layer that routes events to per-session / per-review / per-swarm files |
| `IpcError` | `state/ipc_error.rs` | Structured error envelope every `#[tauri::command]` returns; mirrored in `app/src/lib/ipc.ts` |
| `SwarmState` / `SwarmStatus` | `domain/swarm.rs` | Canonical swarm lifecycle types |
| `Feature` / `FeatureStatus` | `domain/swarm.rs` | Feature with dependencies, status, fix-attempt tracking |
| `ModelSettings` | `domain/swarm.rs` | Which models to use for scout / worker / guard / Hivemind |
| `Milestone` | `domain/swarm.rs` | Group of features with assertions Guard runs |
| `Provider` (trait) | `providers/provider_trait.rs` | Object-safe base trait — every backend implements `call(req: CallRequest) -> Result<ModelResponse>`. Stored in `ProviderRegistry` as `Arc<dyn Provider>`. Impls: `AnthropicProvider`, `AnthropicOAuthProvider` (claude-subscription path using the OAuth token OpenCode stores in `~/.local/share/opencode/auth.json`), `OpenAICompatibleProvider` (OpenAI / Ollama / DeepSeek / any compatible endpoint), `OpenRouterProvider`, plus the test-only `MockProvider`. The streaming variant was split into a separate `StreamingProvider` extension trait (surfaced via `Provider::as_streaming()`) to fix a silent-drop bug where a buffered provider accepted an `mpsc::Sender` and dropped it. Do not merge them back. Only `OpenAICompatibleProvider`, `OpenRouterProvider`, and `MockProvider` implement `StreamingProvider`; both Anthropic impls remain buffered-only. `chatgpt` subscription models have **no** direct provider — they run agentically through OpenCode only. |
| `CallRequest` | `providers/provider_trait.rs:50` | Bundle of args every `Provider::call` takes. Mandatory: `model_id`, `system_prompt`, `user_prompt`. Optional sampling: `temperature`, `top_p`, `max_tokens`, `timeout`. Optional `structured` for tool-use envelopes. `thinking` translates to provider-native reasoning fields (OpenAI `reasoning_effort`, OpenRouter `reasoning.effort`, Anthropic `thinking` budgets). **`cache_static_prefix: bool`** opts the static prefix (system + tools) into Anthropic's `cache_control: ephemeral` markers via `anthropic_system_value` / `anthropic_tools_with_cache` helpers; DeepSeek and other OpenAI-compatible backends ignore the flag (their prefix cache is automatic on byte-stable bodies). Default `false` — only the Nurse classifier opts in today (`nurse/classifier.rs`). |
| `ModelResponse` | `providers/mod.rs:48` | LLM call result. Fields: `output`, `input_tokens`, `output_tokens`, `model_id`, `duration_ms`, plus `cache_hit_tokens` (Anthropic `cache_read_input_tokens` / DeepSeek `prompt_cache_hit_tokens`) and `cache_write_tokens` (Anthropic `cache_creation_input_tokens`; DeepSeek has no equivalent, leaves 0). Providers without prompt caching report both as 0. `derive(Default)` so test fixtures can use `..Default::default()`. |
| `ProviderRegistry` | `providers/mod.rs` | Dispatch + lookup keyed by `provider_id` |
| `ReviewEngine` | `hivemind/engine.rs` | Multi-model review orchestrator |
| `ReviewLogger` | `hivemind/review_log.rs` | Writes the per-review event log under `~/.hyvemind/reviews/{id}.jsonl` |
| `OutputCapture` | `hivemind/output_capture.rs` | Writes per-model full outputs to `~/.hyvemind/reviews/{id}/output-*.txt` |
| `MergeCapture` | `hivemind/merge_capture.rs` | Writes per-merge text to `~/.hyvemind/reviews/{id}/merge-r*.txt` |
| `CircuitBreaker` | `hivemind/circuit_breaker.rs` | 3-state breaker (Closed/Open/HalfOpen) with probe-in-flight gating |
| `ResponseCache` | `hivemind/cache.rs` | moka-based, lock-free, TTL- and size-bounded |
| `AgentHarness` / `HarnessSession` (traits) | `harness/mod.rs` | The runtime-independent trait family. `AgentHarness` is the spawn factory (`id()` / `capabilities()` / `spawn_session(session_id, &SessionOptions, working_dir) -> Result<Arc<dyn HarnessSession>, HarnessError>` / `update_env_vars` / `shutdown`); `HarnessSession` is one live session (send_prompt / steer / abort / force_kill / subscribe_events / is_alive / pid / `classify_event` + `classify_timeout` + `next_recv_timeout` turn classifier). Every trait method returns `HarnessError`. `OpenCodeHarness` is the sole production impl; `MockAgentHarness` covers tests. `BackendSignal` (`Continue` / `TurnComplete` / `TurnError`) is the classifier verdict; `LoopState` is the backend-private per-collect state the host threads through (detectors MUST NOT read it). `HYVEMIND_EXTENSION_TOOLS` (canonical `submit_*` name list) lives here. |
| `HarnessCapabilities` | `harness/capabilities.rs` | `Copy` `u32`-newtype bitset (not `bitflags`), 7 flags: `PROCESS_LIVENESS`, `FORCEFUL_KILL`, `STRUCTURED_TOOL_ARGS`, `SESSION_STATS`, `RETRY_EVENTS`, `RESUMABLE`, `MULTIMODAL_IMAGES`. OpenCode declares `STRUCTURED_TOOL_ARGS \| SESSION_STATS \| RESUMABLE \| MULTIMODAL_IMAGES`; the image flag means OpenCode transport accepts file parts, not that every model supports vision. `all()` is the full set; `all_except_forceful_kill()` is the canonical cooperative-only shape (used by the reduced-cap mock). `names()` produces the flag-name list the Nurse DTO ships to the frontend. |
| `HarnessError` | `harness/error.rs` | The single error currency across the harness surface: `TransportClosed` / `ProcessCrashed { exit_code, stderr }` / `Timeout` / `Unavailable { stderr }` / `SessionNotFound { session_id }` / `SessionExists { session_id }` / `NotSupported(&'static str)` / `Io` / `Json` / `Backend(String)`. `process_health` pattern-matches `ProcessCrashed` for crash detection. |
| `HarnessEvent` / `HarnessSessionStats` | `harness/events.rs` | The runtime-independent event enum (text/thinking deltas, tool start/update/end, lifecycle, auto-retry, plus the `Subagent { tool_call_id, event }` wrapper that routes a task-tool child session's structured activity onto the parent channel) + per-session usage stats. `HarnessSessionStats` billed counters (input / output / reasoning / cache / cost) are SESSION-CUMULATIVE (OpenCode reports per-step usage; the demux accumulates — see `harness/README.md`); the context fields are a latest-snapshot of current context size. Consumers needing per-turn numbers snapshot-then-diff (`commands/chat.rs`). The 100ms / 50-token coalescer that throttles `chat-event` IPC lives in `commands/chat.rs` alongside the chat command. |
| `SessionRuntime` | `harness/session_runtime.rs` | Host-owned per-session runtime — owns the Nurse-critical activity atomics, bounded transcript, owner tag, busy/pinned/needs_respawn flags, captured tool args, `NurseBus` fan-out, cancellation token, and the single event-collection loop. Holds the concrete backend behind `backend: Arc<dyn HarnessSession>`. Construct via `SessionRuntime::wrap(backend, bus, id, token)` (tests: `mock_runtime` / `mock_runtime_with_bus`). |
| `SessionManager` | `harness/session_manager.rs` | The authoritative managed-session registry and **the SOLE strong owner** of every `Arc<SessionRuntime>`. Constructed with `SessionManager::new(Arc<dyn AgentHarness>)`; `spawn(session_id, &SessionOptions, working_dir)` = register-then-publish (`SessionSpawned` carries a `Weak` downgrade); `close` = remove-then-kill (drop the async map guard before `force_kill`). Also drives `close_for_swarm` / `shutdown_all` / `try_evict_one_idle` and holds `SessionStatSnapshot`. |
| `OpenCodeHarness` | `harness/opencode/mod.rs` | The production `AgentHarness` impl. Resolves the user-installed `opencode` binary (`HYVEMIND_OPENCODE_BIN` → PATH → `/opt/homebrew/bin` → `/usr/local/bin`), lazily spawns/respawns the shared `opencode serve` child, persists the Hyvemind→OpenCode session-id map at `~/.hyvemind/opencode/sessions.json` for resume, and receives the full harness env from `AppState::build_harness_env` (provider keys via `config.provider_env_vars()` + the generated `OPENCODE_CONFIG_CONTENT` provider config). Routes scripted sessions (Hyvemind system prompt present) onto the generated `hyvemind-{coder,readonly,guard}` custom agents (`harness/opencode/agents.rs`) so OpenCode's provider base prompt is replaced and tool policy is permission-enforced; promptless Tasks sessions stay on the default `build` agent. |
| `SessionOwner` | `harness/options.rs` | Ownership tag (`Task` / `Review` / `Merge { …, swarm_id }` / `Swarm` / `Unknown`) driving eviction safety (`is_idle_evictable` / `is_reconcile_evictable`). |
| `SessionOptions` / `ThinkingLevel` / `ToolSet` / `HarnessImage` | `harness/options.rs` | Runtime-independent spawn description (`model`, `thinking_level`, `tool_set`, `system_prompt`, `resume`). Constructors `for_model` / `for_scout` / `for_worker` / `for_guard`; builders `with_resume` / `with_thinking_level` / `with_system_prompt` / `with_tool_set`. Model strings stay `provider/model`; the OpenCode backend maps `chatgpt`→`openai` and `claude-sub`→`anthropic` at spawn. All parseable thinking levels (including `max`) are valid — there is no runtime-version validation. |
| `IdleEvictionPolicy` | `harness/eviction.rs` | 10-min idle TTL + 30s maintenance loop over the `SessionManager` registry (SESSION_STATS-gated `needs_respawn` sweep, lock sweep, `session_pool_stats` observability log); emits `session-evicted` |
| `Queen` / `Scout` / `Worker` / `Guard` | `core/{queen,scout,worker,guard}.rs` | Bee-role agents — see `core/README.md` for the loop |
| `Scheduler` | `core/scheduler.rs` | Topological feature ordering with batch dispatch (bounded parallelism) |
| `WorkerHandoff` | `core/handoff.rs` | Structured payload the Worker emits via the `submit_handoff` tool (NOT delimiter-parsed; the host captures tool args off the harness ToolExecutionStart event — the OpenCode echo tool exists only so the tool is registered server-side). Schema: `{ feature_id, run_id, salient_summary, what_was_implemented, verification, success_state: "success"|"failure"|"partial", discovered_issues?: [{severity, description, suggested_fix?}] }`. `feature_id` mismatch against `expected_feature_id` returns `None` from `handoff_from_tool_args` — Queen downcasts on `HandoffParseFailed` to synthesise a Nurse intervention bubble; the feature is marked `Failed`. `SuccessState` deserialiser is case-insensitive with `failed`/`fail` aliases. **There is no fallback parser** — adding one would break the contract. |
| `NurseEngine` | `nurse/engine.rs` | The sole nurse supervisor — subscribes to `NurseBus`, runs the per-session detector registry, hands signals to the `Dispatcher`. `engine.dispatcher: Arc<OnceCell<Arc<Dispatcher>>>` and `engine.in_flight: Arc<Mutex<HashMap<SessionId, DecisionId>>>` are attached before `engine.start()`; `start()` returns `Err` if any required `OnceCell` is empty. Writes the full per-decision chain to `~/.hyvemind/debug/nurse/decisions.jsonl.*` regardless of `HYVEMIND_DEBUG`. **Capability gating**: detectors are capability-agnostic, but the engine SUPPRESSES capability-dependent signals before dispatch when the backing harness lacks the flag — `process_dead` (needs `PROCESS_LIVENESS`; with OpenCode, death falls back to the `Weak`-upgrade-fail `session_gone_unobserved` path), `context_saturation` (needs `SESSION_STATS`; OpenCode has it), `retry_exhaustion` (needs `RETRY_EVENTS`; suppressed on OpenCode). Each session's `harness_id` + `capabilities` flag-name list is captured at registration (defaults to `HarnessCapabilities::all()` when unknown) and re-read on respawn. |
| `Dispatcher` | `nurse/dispatcher.rs` | The three-tier decision pipeline. `Dispatcher::handle_signal` is the single entry point. Tier 1: hardcoded action table for `process_dead` / `crash_pattern` / `session_gone_unobserved` / `no_providers_configured` / `synthesized:process_crashed` / `scheduler_deadlock:*` (no LLM, bypasses storm guard). Tier 2: `SteerPlaybook` templated steer (no LLM). Tier 3: `LlmClassifier` (only path that spends tokens). Holds the `InFlightGuard` (one decision per session at a time), `EventSeq` (monotonic per-decision row counter), `tier1_lookup`, and the `SELF_KILL_GRACE = 30s` window that prevents re-entrant Restart loops. Borrows a `Weak<NurseEngine>` to read snapshots — never re-acquires `engine.sessions`. |
| `DefaultApplier` | `nurse/intervention.rs` | Production `ActionApplier`. Owner-driven routing: `Review` / `Merge` `Cancel` → `cancel_hivemind_review` + best-effort live-session kill; other owners → `mark_self_killed` + `kill_with_verification`. `KillableSession` + `SessionKiller` traits abstract this for testability. Emits the `nurse-event` IPC. |
| `kill_with_verification` | `nurse/intervention.rs` | **Branches on `FORCEFUL_KILL`.** Present (a backend with per-session OS processes) ⇒ the forceful path: abort → 3s grace poll → kill → 7s post-kill poll → `dead_at` row OR `double_fail_giving_up` (no retry). Absent (OpenCode — cooperative-only) ⇒ `cooperative_close_with_verification`: `abort()` → heartbeat-window silence verification → `mark_killed` + cooperative close, or — if events still arrive past the deadline — a `cooperative_close_unverifiable` decision-log stage that **RETAINS** the registry entry (so detectors keep observing the live session) instead of a false forceful kill. An `abort()`-error with no forceful fallback raises a distinct `unkillable_session` stage. |
| `BatchReviewer` | `nurse/batch_review.rs` | Batched periodic LLM review across **all** active sessions: every `HYVEMIND_NURSE_BATCH_INTERVAL_SECS` it renders each session's recent transcript (capped per `HYVEMIND_NURSE_BATCH_EVENTS_PER_SESSION` / `_CHARS_PER_SESSION`), sends ONE provider call via the engine-wide `nurse_model`/`nurse_provider`, parses per-session JSON decisions, and dispatches each through `Dispatcher::dispatch_batch_decision`. Catches stuck/looping sessions the heuristic detectors can't pattern-match. |
| `NurseDecision` | `nurse/snapshot.rs` | Classifier / fast-path output: `LeaveIt { check_back_secs (1-1800) }` / `Steer { message }` / `Restart` / `Cancel { message? }`. **`Restart` in a swarm context typically marks the in-flight feature `Failed`** — use sparingly. Hivemind pseudo-sessions (`source=hivemind`) have no agent session; only `leave_it` and `cancel` are meaningful. Wire shape preserved bit-identically for the frontend. |
| `super_watchdog` | `util/supervise.rs` | Outer panic respawn — respawns the wrapped tokio task exactly once before logging `"nurse unrecoverable"`. Used identically by the session maintenance loop (`lib.rs`) and the Nurse engine's spawned loops; every fire-and-forget supervisor should use this pattern. |
| `NurseBus` | `nurse/bus.rs` | `tokio::sync::broadcast<Arc<NurseBusEvent>>`. Capacity tunable via `HYVEMIND_NURSE_BUS_CAPACITY` (default 4096). `RecvError::Lagged(n)` is logged to `bus.jsonl` and triggers post-lag Tier 2/3 suppression for affected sessions. |
| `NurseBusEvent` | `nurse/bus.rs` | `SessionSpawned { session_id, owner }` / `SessionEnded { session_id }` / `OwnerChanged { session_id, owner }` / `Event { session_id, kind, data }`. Producers (chat, swarm, hivemind) call `touch_activity`-style methods that map to `Event`. |
| `SessionHealth` | `nurse/health.rs` | Per-session detector state — signals ring (capped), tier, escalation state, intervention budget, last-classifier-at. The only engine lock detector code touches. |
| `Signal` / `Severity` / `Tier` | `nurse/health.rs` | `Signal { detector, severity, dedup_key, summary, evidence, raised_at }`. `Severity = Info \| Warn \| Stalled \| Critical`. `Tier = Healthy \| Warning \| Stalled \| Critical`. |
| `Detector` (trait) | `nurse/detector.rs` | `tick(ctx, session) -> Vec<SignalDelta>` + `config_schema() -> Vec<TunableDef>` + `tick_kind() -> Fast \| Slow`. **Per-detector tuning comes from `ctx.profile_config.<field>`** (e.g. `ctx.profile_config.stall`) — never call `StallDetectorConfig::for_profile(ctx.profile)` directly from a detector tick. The engine snapshots `NurseConfig.profile(profile)` once per tick (`engine.rs:run_periodic_sweep` / `on_session_event` / SessionEnded branch) so user edits via `set_nurse_profile` take effect on the next tick without a restart. Lint-enforced via integration test: every raised `Signal` must populate `evidence` with everything a reader needs to second-guess the raise — *"if a future Claude Code session 30 days from now would have to read source to understand why this signal raised, evidence is incomplete"*. |
| `DetectorRegistry` | `nurse/detector.rs` | Holds the seven built-in detectors and dispatches them per `TickKind`. Slow detectors run on a separate task to keep the engine loop unblocked. |
| `StallDetector` / `ReasoningLoopDetector` / `ToolFailureDetector` / `ProcessHealthDetector` / `ProviderHealthDetector` / `ContextSaturationDetector` / `RetryExhaustionDetector` | `nurse/detectors/*.rs` | The seven built-ins. Stall = time-based + post-prompt silence. ReasoningLoop = siphash exact + compression + minhash paraphrase. ToolFailure = repeated tool-call failures clustered by signature. ProcessHealth = backend liveness (PROCESS_LIVENESS-gated) + stderr crash patterns. ProviderHealth = circuit-breaker / missing-provider. ContextSaturation = context-window % threshold (SESSION_STATS). RetryExhaustion = auto-retry-end clustering in a sliding window (RETRY_EVENTS — suppressed on OpenCode). |
| `SteerPlaybook` | `nurse/playbook.rs` | Tier 2 templated steer table. Matches by `dedup_key` substrings and returns a canned message — no LLM call required. |
| `StormGuard` | `nurse/storm_guard.rs` | Per-`(session_id, kind)` sliding-window guard. Default 3 events / 60s window before gating. **Tier 1 bypasses storm guard** (deterministic actions are always allowed). |
| `BudgetState` | `nurse/budget.rs` | Per-detector + age-decay intervention budget. Initial cap + per-hour decay + max cap + per-detector cap + per-`dedup_key` cooldown. Defaults per profile in `ProfileConfig::default_for(profile)`. Derives `Clone`; `is_cooldown_elapsed` is a pure-read helper safe to call from the dispatcher snapshot path. |
| `LlmClassifier` | `nurse/classifier.rs` | Tier 3 wrapper around `ProviderRegistry`. Reads `NurseConfig::effective_model(profile)` / `effective_provider(profile)` (per-profile override with engine-wide fallback); returns `Skip` when no model configured (logged as `classifier_skipped_no_model` in the decision chain). 90s provider-call timeout via `HYVEMIND_NURSE_PROVIDER_TIMEOUT_SECS`. |
| `ObservabilityHandles` | `nurse/observability/mod.rs` | Bundle of always-on writers: `DecisionLogger`, `ClassifierCapture`, `SignalStream`, `BusTelemetry`. Constructed once at engine init. `prune_on_startup` enforces retention by mtime. |
| `DecisionLogger` | `nurse/observability/decision_log.rs` | Daily-rotated `decisions.jsonl.YYYY-MM-DD` writer. Emits the full decision-chain event sequence (decision_started / storm_guard_evaluated / tier1_evaluated / playbook_evaluated / classifier_invoked / classifier_returned / intervention_dispatched / kill_verification / intervention_outcome / decision_finalised). Retention via `HYVEMIND_NURSE_DECISION_LOG_RETENTION_DAYS` (default 30). |
| `ClassifierCapture` | `nurse/observability/capture.rs` | Writes `captures/{decision_id}-prompt.txt` and `-response.txt` for every Tier 3 invocation. Prompt is written BEFORE the provider call so a crash mid-call leaves an unambiguous "invoked, never returned" signal on disk. 1 MiB cap via `HYVEMIND_NURSE_CAPTURE_MAX_BYTES`. |
| `SignalStream` | `nurse/observability/signal_stream.rs` | Per-session `signals/{session_id}.jsonl` with `.1`/`.2`/`.3` rotation at 4 MiB (`HYVEMIND_NURSE_SIGNAL_STREAM_MAX_BYTES`). Pruned 24h after `SessionEnded`. |
| `BusTelemetry` | `nurse/observability/bus_telemetry.rs` | `bus.jsonl.YYYY-MM-DD` — bus-level lifecycle (SessionSpawned / SessionEnded / OwnerChanged) + `Lag(n)` + `capacity_pressure` (sampled 1/s when broadcast buffer crosses 80%) + `post_lag_suppression_entered`/`_exited`. Retention via `HYVEMIND_NURSE_BUS_LOG_RETENTION_DAYS` (default 30). |
| `NurseConfig` | `nurse/config.rs` | Master-level switches: `enabled`, `mode: NurseMode (Enabled \| Observe \| Disabled)`, `nurse_model`, `nurse_provider`, `profiles: HashMap<NurseProfile, ProfileConfig>`. `effective_model(profile)` / `effective_provider(profile)` resolve per-profile override with engine-wide fallback. Maps onto top-level `config.json` fields. |
| `ProfileConfig` | `nurse/config.rs` | Per-context tuning: per-detector configs, `intervention_mode (Auto \| Observe)`, `budget`, `escalation_min_severity`, plus optional per-profile `nurse_model` / `nurse_provider: Option<String>` that override the engine-wide classifier. Defaults per `NurseProfile::default_for(profile)`. |
| `NurseProfile` | `nurse/config.rs` | `Tasks \| Swarm \| Hivemind \| Test \| Default`. `for_owner(&SessionOwner)` selects the profile from the owner kind. |
| `SynthesizedKind` / `InterventionOwner` / `describe_synthesized` | `nurse/synthesized.rs` | Error cases that don't originate from a live agent session (CircuitBreakerOpen, ProtocolViolation, SchedulerDeadlock, SteerFailed). `describe_synthesized` table maps each kind to canned summary + severity; adding a kind requires extending the table (no implicit default). |
| `TunableDef` | `nurse/snapshot.rs` | Per-tunable metadata (`unit`, `direction`, `default`, `safe_range`, markdown `description`) returned by `Detector::config_schema()`. Drives the auto-generated Profile-tab UI so a new detector ships with its tuning controls without React changes. |

## Bee-role tool & thinking-level matrix

Per-role spawn wiring lives in `harness/options.rs` (`SessionOptions::for_scout/for_worker/for_guard`). Per-swarm overrides via `model_settings.{role}_thinking_level`. Role values accept `off|minimal|low|medium|high|xhigh|max` — any parseable level is valid (there is no runtime-version validation). The OpenCode backend translates `ToolSet` into a per-message tool allowlist (`harness/opencode/tools.rs`: `find`→`glob`, `ls`→`list`; read-only = `read`/`grep`/`glob`/`list`/`webfetch`/`websearch` with the mutating set denied; full coding = no restriction).

| Role | Thinking (default) | Tool set | Structured-output tool | Failure mode if tool is never called |
|------|--------------------|----------|---------------------------|-------------------------------------|
| Queen | `Medium` | Full coding + bash (runtime persona); read-only (planning persona) | `submit_features` (planning) | Feature decomposition errors |
| Scout | `High` | Read-only (`read`/`grep`/`glob`/`list`/`webfetch`/`websearch`); **no bash** | `submit_scout_result` | `"scout for feature '<id>' did not call submit_scout_result"` (`scout.rs`) → feature `Failed` |
| Worker | `Medium` | Full coding + bash (OpenCode's full built-in set, incl. its `task` subagent tool) | `submit_handoff` | `HandoffParseFailed` → Queen downcast → Nurse bubble → feature `Failed` |
| Guard | `Medium` | Custom: `read` + `bash` + `grep` + `glob` + `list` (+ all `submit_*` tools) | `submit_guard_result` | `"guard for validator feature '<id>' did not call submit_guard_result"` (`guard.rs`) |
| Nurse classifier | `Low` | **None** (pure classifier; non-streaming `ProviderRegistry.call`) | `nurse_decisions` (schema: `nurse/schema.rs`) | `consecutive_bad_parse_ticks` increments; loop continues. Dispatch lives in `nurse/dispatcher.rs`; Tier 3 LLM wrapped by `nurse/classifier.rs`. |
| Nurse detectors | — (no LLM) | **None** (pure heuristics over `SessionHealth`) | — (detectors raise `Signal`s; no tool boundary) | Detector tick error → counted in detector stats; engine loop continues |

**Nurse has two surfaces**: detectors (pure heuristics, run every tick) and the three-tier dispatcher. Only Tier 3 (LLM classifier) spends tokens — Tier 1 (deterministic) and Tier 2 (templated playbook) fire without a model call.

**There is no fallback parser for any `submit_*` tool** — the Rust backend captures `args` directly off the harness ToolExecutionStart event (`STRUCTURED_TOOL_ARGS`). The generated OpenCode echo tools are deliberate no-ops: they exist only so the tool names are registered server-side and the model can call them. Do not add regex fallbacks "just in case" — it would defeat the contract and mask prompt regressions.

**`submit_*` echo tools are generated at startup** by `harness/opencode/tools.rs` — one no-op echo tool file per `HYVEMIND_EXTENSION_TOOLS` entry, written into the server cwd (`~/.hyvemind/opencode/.opencode/tool/`) AND the OpenCode global config dir (`$XDG_CONFIG_HOME/opencode/tool/`, default `~/.config/opencode/tool/`). The global copy is the one that reaches every session (sessions are created against project directories, not the server cwd); registration is verified per directory via `GET /experimental/tool/ids`. Subagent delegation: only the Worker (full tool surface) gets OpenCode's built-in `task` tool — Scout's read-only allowlist and Guard's custom allowlist exclude it. There are no `subagent`/`mcp` extension tools.

## Hidden invariants & lock ordering

Rules that are not visible from reading one file at a time. Breaking any of them is a real bug — most have already been broken once.

### Cross-subsystem invariants (domain layer)

- **`domain/` imports nothing from `core::*`, `state::*`, or `harness::*`** (`domain/mod.rs:1`). It exists to break the historical `core` ↔ `state` cycle; reintroducing such an import re-creates the cycle.
- **`Feature.status == Completed` MUST be paired with a persisted `WorkerHandoff`** at `~/.hyvemind/swarms/{id}/handoffs/{feature_id}.json`. Exception: `validate-*` features (validators have no Worker step). Marking Completed without a handoff breaks Queen replay.
- **The crash reconciler is the ONLY writer of `SwarmStatus::Interrupted`** and the ONLY setter of `Feature.interrupted = true && Feature.resumable = true` (`core/recovery.rs:81,366`). All three are cleared by `resume_swarm`. Setting them anywhere else corrupts the Resume affordance.
- **`max_fix_attempts` is forward-only** (default `3`, `domain/swarm.rs:355`). Removing the check at `core/queen.rs:1804` lets Guard spawn fix-features indefinitely.
- **`Milestone.sealed` is forward-only** (`true → false` is never valid). The scheduler treats sealed milestones as closed worlds (`core/scheduler.rs:100`).
- **`SwarmState.queen_plan_review_done` is forward-only** — once a Queen master-plan Hivemind review is attempted, it's suppressed for the swarm's lifetime (clones / resumes carry it forward).
- **`Feature::is_validator()` is keyed on the `validate-` id prefix** (`domain/swarm.rs:365`). Validator features skip Scout/Worker. Renaming the prefix without updating the predicate silently routes them through the wrong pipeline.
- **Every `#[serde(default)]` on a domain type is load-bearing** — each matches a real file format on user disks. Removing one breaks back-compat for older `state.json` / `features.json` files.

### Lock ordering & async discipline

- **Nurse `config` → `sessions`**: `engine.config: tokio::RwLock` is acquired and released BEFORE `engine.sessions: std::sync::RwLock`. Never hold the sync `sessions` guard across an `.await`, and never call `SessionManager` methods or dispatch interventions while holding it. The dispatcher enforces this by snapshotting `nurse_cfg` before each sessions-guarded block.
- `features: Arc<RwLock<Vec<Feature>>>` is **dominant** over `swarm_state: Arc<RwLock<SwarmState>>` (`core/queen.rs:7`). Hold neither across `.await` unless the clone-and-drop idiom from that header comment is used.
- `SwarmUsageAccumulator` wraps `std::sync::Mutex` — **sync-only, never `.await`-spanning**. Lock-poison policy is `unwrap_or_else(|e| e.into_inner())` (project-wide for additive counters; `domain/swarm.rs:11`).
- `ProviderRegistry` sits behind an `AsyncRwLock`. Callers `Arc::clone` the result from `registry.get(name)` and **drop the read guard before any I/O** — a refresh write lock blocks until in-flight reads drop.

### Session ownership (the harness single-owner invariant)

- **`SessionManager`'s registry is the SOLE strong owner of every managed `Arc<SessionRuntime>`** (`harness/session_manager.rs`), which is in turn the sole strong owner of the inner `Arc<dyn HarnessSession>`. Nurse holds only `Weak<SessionRuntime>`; `AgentHarness` impls (the `OpenCodeHarness` factory) hold no per-session strong ref. Re-introducing a second strong holder would defeat the single-owner model and resurrect the lifecycle races the abstraction fixed — `Weak::upgrade()` failing is exactly how `session_gone_unobserved` death detection works.
- `SessionManager::close` is **remove-then-kill**: take the async map lock only long enough to `remove` the entry, drop the guard, THEN `mark_killed` + `force_kill().await` on the locally-owned handle. **Never hold the registry map lock across `force_kill`.** The cooperative `cooperative_close_unverifiable` path deliberately **retains** the registry entry (the session is still emitting events past the deadline) — see the Nurse section. On OpenCode, the cooperative `force_kill` **retains the server-side session** so resume stays possible.

### Provider registry refresh contract

- Every command that mutates `config.providers` or `config.provider_keys` **MUST call `AppState::refresh_provider_registry`** (`state/app_state.rs:624`). Forgetting it leaves Hivemind / Nurse dispatching against a stale snapshot until app restart. `save_api_key`, `delete_api_key`, `add_provider` all do this — new mutators must too.
- Every `update_env_vars` push to the harness **MUST be built via `AppState::harness_env` / `build_harness_env`**, never bare `config.provider_env_vars()` — the bare map lacks the generated `OPENCODE_CONFIG_CONTENT`, so the next `opencode serve` spawn would silently lose every auto-registered custom provider.

### Extension contract (provider-usage pollers)

- **Never construct your own `reqwest::Client`** — use `ctx.http()` (shared pool, 30s timeout).
- **Never hold a config read guard across `.await`** in `fetch()` — read into a local, drop, then I/O.
- **`ExtensionError::Unsupported` is terminal** — the poller exits permanently. Use `Auth`/`Network`/`Parse`/`Internal` for transient errors (auto-backoff applies).
- **`MIN_REFRESH_INTERVAL_SECS = 30`** (`extensions/poller.rs:38`); lower values are silently clamped.

## Frontend Architecture

The frontend lives under `app/src/`. It's a React 18 + Vite + Tailwind shell speaking to the Rust backend exclusively through Tauri IPC (`invoke()` for commands, `listen()` for events). All LLM provider HTTP traffic happens in Rust — the renderer never reaches an external host.

### Stack

| Layer | Version / config | Notes |
|-------|-----------------|-------|
| UI runtime | React 18.3, TypeScript strict | No SSR; everything renders in the Tauri WebView |
| Bundler | Vite 4 on port `1430` (`strictPort: true`, HMR disabled) | HMR disabled to avoid Tauri reload races; the dev shell rebuilds on file save instead |
| CSS | Tailwind 3.4 (custom palette — see Design system below) | No CSS-in-JS, no shadcn — bespoke `components/atoms.tsx` library |
| Routing | Custom `ScreenRouter` in `App.tsx` (no react-router) | `NavState { tab, params }`; Cmd/Ctrl + 1-6 jumps between tabs |
| State | React Context + custom hooks (no Redux / Zustand) | `TaskRuntimeProvider` is the heavyweight; see State management below |
| IPC wrapper | `lib/ipc.ts` (typed `invoke<T>` around `@tauri-apps/api/core`) | Sentry capture on every call + `IpcError` discriminator |
| Tests | Vitest 4 + Testing Library + jsdom | ~75 test files under `**/__tests__/` (758 tests); polyfills in `test/setup.ts` |
| Error tracking | `@sentry/react` 10 + `tauri-plugin-sentry-api` | Renderer ships events via the Tauri Sentry plugin (not direct HTTP) |
| DnD | `@dnd-kit/core` + `sortable` + `modifiers` | Topbar pill reorder + task list sorting |
| Markdown | `react-markdown` 10 + `remark-gfm` | Strict — no `rehype-raw`; HTML is escaped (CSP relies on this) |

### Directory layout

```
app/src/
  App.tsx                 # Root: provider tree, Topbar, Sidebar, ScreenRouter, keyboard shortcuts
  main.tsx                # React DOM entry point + Sentry init
  index.css               # Tailwind layers + custom utilities (hex-bg, shimmer, pulse-*, honey-edge)
  A11Y.md                 # Frontend accessibility conventions (aria-live, screen-reader rules)
  screens/                # Top-level routable views
    __tests__/
  components/             # Shared UI library (atoms + composed widgets)
    __tests__/
  lib/                    # IPC wrapper, context providers, event stores, runtime
    __tests__/
  hooks/                  # Custom React hooks (useNurseStatus, etc.)
  types/                  # IPC payload + domain TypeScript types
  extensions/             # Topbar extension widgets (provider usage pills)
    __tests__/
  state/                  # TestRunProvider lives here
  data/                   # Mock data for browser-preview / Storybook-style use
  test/                   # Vitest setup (polyfills)
```

### Screens

13 screens, registered in `App.tsx` `ScreenRouter`. All accept `go: GoFn` and optionally screen-specific params.

| Tab | File | Purpose |
|-----|------|---------|
| `landing` | `screens/Landing.tsx` | Public-facing product landing page inside the app shell; not locked behind the OpenCode runtime gate |
| `dashboard` | `screens/Dashboard.tsx` | Stats overview + cost trends |
| `tasks` | `screens/Tasks.tsx` | Planning conversation (the only conversational surface) |
| `swarms` | `screens/Swarms.tsx` | Swarm list + management |
| `new-swarm` | `screens/NewSwarm.tsx` | Swarm creation / edit form |
| `swarm-control` | `screens/SwarmControl.tsx` | Live swarm execution monitor |
| `hiveminds` | `screens/Hiveminds.tsx` | Multi-model review templates list |
| `hivemind-edit` | `screens/HivemindEdit.tsx` | Rounds + models + orchestrator config |
| `model-browser` | `screens/ModelBrowser.tsx` | Searchable model catalog |
| `review-history` | `screens/ReviewHistory.tsx` | Historical review runs for a Hivemind |
| `tests` | `screens/Tests.tsx` | Stability-test runner + results |
| `nurse` | `screens/Nurse.tsx` | Nurse Hive UI — 4 internal tabs: Live Sessions, Intervention Log, Detector Activity, Profiles. In `LOCKED_WHEN_NO_RUNTIME` (Nurse without an agent runtime is meaningless). |
| `settings` | `screens/Settings.tsx` | API keys, providers, zoom, sound, source dir, master Nurse toggle, OpenCode runtime status card (installed badge + binary path + version + install hint + "Open OpenCode" button) |

**To add a new screen**: write the file under `screens/`, import it in `App.tsx`, add a case to `ScreenRouter`, add a tab entry to `NAV` if it's a top-level destination, and (if hidden behind the runtime gate) add the tab id to `LOCKED_WHEN_NO_RUNTIME`. Then update §Screens in this section.

### State management

No Redux. Context providers nest in `App.tsx`; each owns one slice of state and subscribes to the Tauri events that drive it.

| Provider | File | Owns | Subscribes to |
|----------|------|------|---------------|
| `SettingsProvider` | `lib/SettingsProvider.tsx` | Backend `SettingsResponse` cache; `useSettings()` / `useSetting<K>()` hooks | `default-model-changed`, `default-project-path-changed`, `default-hivemind-changed` |
| `ProvidersProvider` | `lib/ProvidersProvider.tsx` | Provider list + configured status | `usage-snapshot-updated` (debounced 250ms) |
| `TaskRuntimeProvider` | `lib/taskRuntime.tsx` (~5,400 LOC) | Six internal context slices (`taskRuntime.tsx:835-840`): `TaskDraftContext` (in-flight prompt), `TaskListContext` (per-task message arrays), `TaskRuntimeStateContext` (active session / streaming), `TaskActionsContext` (send/stop/resume callbacks), `HivemindOptionsContext` (review-tier selection), `DefaultsContext` (model + project defaults) | `chat-event`, `hivemind-progress`, `swarm-activity`, `nurse-event` |
| `ExtensionProvider` | `extensions/ExtensionProvider.tsx` | Topbar pill order (localStorage) + usage snapshots | `usage-snapshot-updated` |
| `TestRunProvider` | `state/TestRunProvider.tsx` | Active test-run state | `test-progress` |
| `ErrorModalProvider` | `components/ErrorModal.tsx` | Global error surface + Sentry capture | — |
| `ToastProvider` | `components/Toast.tsx` | Toast queue | — |
| `ContextMenuProvider` | `components/ContextMenu.tsx` | Right-click menu state | — |
| `ProjectContext` | `App.tsx` + `components/ProjectPicker.tsx` | Project list + active project (localStorage) | — |
| `RuntimeGate` (`useRuntimeGate`) | `App.tsx` | OpenCode-installed gate for navigation (driven by `getOpencodeStatus().installed`); locks dashboard / tasks / swarms / new-swarm / swarm-control / hiveminds / hivemind-edit / model-browser / review-history / nurse when the runtime is missing (`LOCKED_WHEN_NO_RUNTIME`) | — |
| `NurseProvider` | `lib/NurseProvider.tsx` | Engine status snapshot + live sessions + intervention log paging cache. Drives the new `nurse` screen's four tabs. | `nurse-event` (via `nurseEventStore`) |

### IPC layer (`lib/ipc.ts`)

Single chokepoint wrapping `@tauri-apps/api/core::invoke`. Every call:

1. Forwards to `rawInvoke<T>(name, args)` (forwards name-only when `args === undefined` to keep test contracts simple).
2. On rejection, capture to Sentry with tags `source: "ipc"`, `ipc_command`, and (when present) `ipc_error_kind`.
3. Re-throw so callers' error handling (`console.error` → `ErrorModal`) is unchanged.

The `IpcError` discriminator mirrors `state/ipc_error.rs`:

| `kind` | When |
|--------|------|
| `provider_unauthenticated` | API key missing / rejected |
| `provider_rate_limited` | 429 from upstream |
| `circuit_breaker_open` | Per-provider breaker tripped |
| `not_found` | Resource (with `resource` + `resource_id` payload) missing |
| `validation` | Request didn't pass validation |
| `not_approved` | User hasn't approved the working directory (`request_working_dir_approval`) |
| `internal` | Anything else |

`formatIpcError(err)` turns any rejection (typed envelope, `Error`, or string) into a user-facing string for toasts / modals. Use it at every error display site.

### Event listener index

These are the consumer-side counterparts to the §Tauri Events backend table. Mirroring this so a frontend agent can find "who listens for X" without grepping the whole tree.

| Event | Subscribed in | Use |
|-------|--------------|-----|
| `chat-event` | `lib/taskRuntime.tsx` | Streams Tasks tokens, drives Tasks completion |
| `hivemind-progress` | `lib/hivemindEventStore.ts` (singleton fan-out) + `lib/taskRuntime.tsx` | Routes per-review events to the right consumer (task vs swarm vs standalone) |
| `swarm-event` | `lib/taskRuntime.tsx`, `screens/Swarms.tsx`, `screens/SwarmControl.tsx` | Debounced state refresh on lifecycle change |
| `swarm-activity` | `lib/swarmActivityStore.ts` (singleton fan-out) + `screens/SwarmControl.tsx` | Per-feature agent activity stream. On first subscribe for a swarm the store hydrates from `get_swarm_activity_log` (paged, deduped against live events via per-event `seq`) so SwarmControl can replay history before live events take over. |
| `swarm_reconciled` | `screens/Swarms.tsx` | Render Resume badges immediately after a startup-replay reconciliation |
| `nurse-event` | `lib/nurseEventStore.ts` (singleton fan-out) + `lib/taskRuntime.tsx`, `hooks/useNurseStatus.ts`, `lib/NurseProvider.tsx` | Status snapshot + in-flow intervention bubbles. Promoted to a singleton store with the Nurse screen — multiple consumers (tasks runtime + Nurse provider + hook) must not each `listen()` against the channel or every event doubles. |
| `test-progress` | `state/TestRunProvider.tsx`, `screens/Tests.tsx` | Test phase + result updates |
| `usage-snapshot-updated` | `extensions/ExtensionProvider.tsx`, `lib/ProvidersProvider.tsx` | Topbar pill refresh + provider-configured status |
| `default-model-changed` / `-project-path-changed` / `-hivemind-changed` | `lib/SettingsProvider.tsx` (+ `lib/taskRuntime.tsx`) | Patches the cached `SettingsResponse` |
| `session-evicted` | `lib/taskRuntime.tsx` | Marks an idle-evicted session in the Tasks runtime (a synthetic `chat-event` notice accompanies it) |

**Singleton event stores** — `lib/hivemindEventStore.ts` and `lib/swarmActivityStore.ts` each register **one** Tauri listener and fan events out to per-id callbacks via `subscribe(reviewId, cb)` / `subscribe(swarmId, cb)`. **Opening a second `listen()` against the same channel doubles every event** — the Tauri runtime does NOT deduplicate. The moment a second consumer appears for a channel, promote it to a store.

`swarmActivityStore` additionally **hydrates on first subscribe per swarm**: pages `get_swarm_activity_log(swarmId, afterSeq)` up to 200 pages, folds via `applyActivityEvent`, tracks `maxSeenSeq`. Live events received during hydration are buffered and **deduped against the log via the per-event `seq` field** when hydration completes. LRU bookkeeping caps at `MAX_SWARMS = 50` (`swarmActivityStore.ts:13`); oldest evicts when over the cap. Model new stores on this pattern when paged history + dedup is needed.

### Design system

Custom Tailwind palette in `app/tailwind.config.ts`. The whole dark theme is "ink + honey":

| Token | Value | Use |
|-------|-------|-----|
| `ink-950` | `#07080C` | Deepest background |
| `ink-900` | `#0B0D12` | Body background |
| `ink-850` / `ink-800` / `ink-700` | `#0F1219` / `#13161F` / `#1A1E2A` | Panel / surface backgrounds (lightest → darkest panel) |
| `ink-600` / `ink-500` / `ink-400` | `#222837` / `#2A3142` / `#3A4254` | Borders, dividers, raised surfaces |
| `honey-500` | `#F5B919` | Primary accent (buttons, active nav, wordmark) |
| `honey-50` … `honey-900` | full scale | Tints and shades |
| `line` / `line-soft` / `line-strong` | `#1F2433` / `#171B26` / `#2C3346` | Border palette |
| `muted` | `#8B92A5` | Disabled text |
| `dim` | `#5A6072` | Secondary text |

Custom shadows: `glow` (honey ring + drop), `panel` (subtle inset). Fonts: `Inter` (sans), `JetBrains Mono` (mono).

Custom CSS utilities in `index.css`: `.hex-bg` (animated hex tile background), `.shimmer` (token-streaming shimmer), `.pulse-{green,amber,cyan,brain}` (status dot pulses), `.honey-edge` (double-border inset), `.card-hover` (lift + border + shadow on hover).

### Accessibility

See `app/src/A11Y.md` for the complete convention guide. Quick summary:

- Use `aria-live="polite"` for status updates the user shouldn't be interrupted by (token streaming, background sync)
- Use `aria-live="assertive"` only for error toasts and modal alerts
- Streaming text is **non-atomic** (`aria-atomic={false}`) so each chunk is announced incrementally
- Manual VoiceOver pass required for any feature touching the chat stream, swarm activity, or modal flow — there is no automated SR test
- Focus traps live in modals via `focus-trap-react`

## Diagnosing Hyvemind entities

**When given an opaque ID — a session UUID (full or partial), task number (`306` / `task-306`), swarm UUID, `hmr-…` review id, or nurse `decision_id` — and asked to investigate it, do NOT start with the manual recipes below.** Run the bundled diagnosis script first (it's also exposed as the `hyvemind-diagnose` project skill in `.claude/skills/hyvemind-diagnose/`):

```bash
python3 .claude/skills/hyvemind-diagnose/diagnose.py <ID>
```

One call auto-detects the entity type and prints the entire diagnostic bundle: known-failure-pattern scan (start interpreting from `DETECTED FAILURE PATTERNS`), condensed timeline, all WARN/ERRORs, nurse signals + decisions, captured tool args, and linked entities (a task auto-deep-dives into its most recent session; a nurse decision pulls in the session it ran on). Flags: `--full` disables truncation caps; `--type session|task|swarm|review|decision` skips auto-detection for ambiguous prefixes. Every truncated section prints the exact file path holding the rest — read that path directly instead of re-running broader commands.

The skill's `SKILL.md` carries the failure-pattern interpretation table and the expected `submit_*` tool-arg shapes. The manual per-entity recipes below are the **fallback** for when the script can't resolve an ID or you need raw access to a specific log surface.

## Investigating a Task

Task messages (frontend UI state) are stored as JSON arrays in `~/.hyvemind/task-messages/task-{NUMERIC_ID}.json`. Each element is a message object with `who` (user/asst/plan/questions/session-divider), `text`, `tools`, `reasoning`, `model`, etc.

```bash
# List all task message files
ls -lt ~/.hyvemind/task-messages/

# Read a task's messages (pretty-printed)
python3 -m json.tool ~/.hyvemind/task-messages/task-{ID}.json

# Find which task contains a specific session ID
grep -l '{SESSION_ID}' ~/.hyvemind/task-messages/*.json

# Summarize task messages (who, text preview, tool count)
python3 -c "
import json, sys
with open(sys.argv[1]) as f:
    for i, m in enumerate(json.load(f)):
        who = m.get('who','?')
        text = (m.get('text','') or '')[:100].replace(chr(10),' ')
        tools = len(m.get('tools',[])) if m.get('tools') else 0
        plan = bool(m.get('planText'))
        print(f'[{i:2d}] {who:20s} tools={tools} plan={plan} {text}')
" ~/.hyvemind/task-messages/task-{ID}.json
```

## Investigating a Session ID

When given a session ID to investigate, the primary surfaces are the per-session **debug log** (`~/.hyvemind/debug/sessions/{session_id}.jsonl`, written when `HYVEMIND_DEBUG=1`) and the **Nurse observability** files (`~/.hyvemind/debug/nurse/signals/{session_id}.jsonl` + the decision log — always on). Hyvemind no longer writes its own session transcripts; the conversation itself lives server-side in OpenCode (resumable via the id map at `~/.hyvemind/opencode/sessions.json`).

```bash
# Per-session debug log (everything in this file is for one session)
cat ~/.hyvemind/debug/sessions/{SESSION_ID}.jsonl | python3 -m json.tool

# Last N events, summarized
rtk proxy tail -100 ~/.hyvemind/debug/sessions/{SESSION_ID}.jsonl | python3 -c "
import json, sys
for line in sys.stdin:
    try:
        obj = json.loads(line.strip())
        ts = obj.get('timestamp','')[:19]
        lvl = obj.get('level','')
        msg = obj.get('fields',{}).get('message','')
        print(f'{ts} [{lvl}] {msg}')
    except: pass
"

# Nurse signal stream for the session (always on — no HYVEMIND_DEBUG needed)
cat ~/.hyvemind/debug/nurse/signals/{SESSION_ID}.jsonl | python3 -m json.tool

# Nurse decisions that touched the session
jq -c "select(.session_id == \"{SESSION_ID}\")" ~/.hyvemind/debug/nurse/decisions.jsonl.*

# Which OpenCode server-side session does this id map to?
python3 -m json.tool ~/.hyvemind/opencode/sessions.json | grep -A1 '{SESSION_ID}'

# Find which task contains the session ID
grep -l '{SESSION_ID}' ~/.hyvemind/task-messages/*.json
```

**Legacy transcripts**: `~/.hyvemind/chat-sessions/{session_id}.jsonl` files are no longer written or read by the app. They may still be useful when investigating an old (pre-OpenCode) session, but no new files appear there.

## Investigating a Hivemind Review

When given a review ID (e.g., `hmr-a1b2c3d4`):

- High-level event log (always written when `HYVEMIND_DEBUG=1`): `~/.hyvemind/reviews/{review_id}.jsonl`
- Per-model full outputs (capture files, written alongside the event log): `~/.hyvemind/reviews/{review_id}/output-{model_id_safe}-r{round}.txt` (model id with `/` and `:` replaced by `_`)
- TRACE-level firehose (provider request/response, internal state): `~/.hyvemind/debug/reviews/{review_id}.jsonl`
- SQLite job records: `~/.hyvemind/hivemind/`

The event log no longer inlines the full `output` string. `model_call_completed` events carry `output_file` (relative path under `~/.hyvemind/`) and `output_len` instead — `cat` the referenced file for the full text.

### Commands

```bash
# Read the high-level event log (pretty-printed)
cat ~/.hyvemind/reviews/hmr-a1b2c3d4.jsonl | python3 -m json.tool

# Read the TRACE-level firehose for the same review
cat ~/.hyvemind/debug/reviews/hmr-a1b2c3d4.jsonl | python3 -m json.tool

# List all review event logs sorted by most recent
ls -lt ~/.hyvemind/reviews/

# List per-model output captures for one review
ls -lt ~/.hyvemind/reviews/hmr-a1b2c3d4/

# Summarize review timeline
python3 -c "
import json, sys
with open(sys.argv[1]) as f:
    for line in f:
        obj = json.loads(line.strip())
        ts = obj.get('timestamp','')[:19]
        evt = obj.get('event','')
        d = obj.get('data',{})
        keys = ['model_id','provider','round','session_id','error']
        info = {k: d[k] for k in keys if k in d}
        print(f'{ts} [{evt}] {info}')
" ~/.hyvemind/reviews/hmr-a1b2c3d4.jsonl

# Extract model outputs from a review (cat the capture files)
python3 -c "
import json, sys, os
home = os.path.expanduser('~/.hyvemind')
with open(sys.argv[1]) as f:
    for line in f:
        obj = json.loads(line.strip())
        if obj.get('event') == 'model_call_completed':
            d = obj['data']
            m = d.get('model_id','?')
            tok = f\"{d.get('input_tokens',0)} in / {d.get('output_tokens',0)} out\"
            dur = f\"{d.get('duration_ms',0)}ms\"
            print(f'\\n--- {m} ({tok}, {dur}) ---')
            of = d.get('output_file')
            if of:
                with open(os.path.join(home, of)) as outf:
                    print(outf.read()[:500])
" ~/.hyvemind/reviews/hmr-a1b2c3d4.jsonl

# Check for errors
grep -i 'failed\|error' ~/.hyvemind/reviews/hmr-a1b2c3d4.jsonl | python3 -m json.tool

# Find a review by partial ID
find ~/.hyvemind/reviews/ -name "hmr-a1b2*"
```

### Review log events

| Event | When | Key Data |
|-------|------|----------|
| `context_started` | Context-gather agent session created | session_id |
| `context_completed` | Context gathered | enriched prompt length |
| `engine_started` | Backend begins model dispatch | models, stance, timeout |
| `round_started` | Round begins | round number, model count |
| `model_call_started` | Model dispatched | model_id, provider |
| `model_call_completed` | Model responds | `output_file` (relative path), `output_len`, tokens, cost, duration |
| `model_call_failed` | Model errors | error message |
| `round_completed` | All models done | response count |
| `merge_started` | Merge agent session created | session_id, round |
| `merge_completed` | Merge produces updated plan | round |
| `review_completed` | All rounds done | total_rounds |

### Cross-referencing agent sessions

Context and merge phases run as OpenCode-backed harness sessions. The review log records their `session_id` values; cross-reference them via the per-session debug log:
```bash
cat ~/.hyvemind/debug/sessions/{SESSION_ID}.jsonl | python3 -m json.tool
```

## Investigating a Nurse decision

`~/.hyvemind/debug/nurse/` is always-on (NOT gated on `HYVEMIND_DEBUG`). Every Nurse decision — whether it ends in a dispatched intervention, a gated drop, or a no-op — is keyed by a `decision_id` (uuid4 simple). All chain events for one decision share that id. The dispatcher in `nurse/dispatcher.rs` is the single source of truth: `decisions.jsonl.*` records both what fired and *why*. The `nurse-event` IPC + `progress_log.jsonl` `nurse_intervention` rows record the user-visible action.

### On-demand bait — `Test Nurse ▾` dropdown

To fire any detector live without waiting for the wild to produce one, open the Tasks composer and click **Test Nurse ▾** (next to the Auto button). The catalog lives in `app/src/lib/nurse-scenarios.ts` and the dropdown is `app/src/components/NurseTestDropdown.tsx`. All five scenarios (stall / reasoning-loop-exact / paraphrase-loop / tool-failure-burst / context-saturation) fill the composer with a bait prompt. Watch interventions appear inline via `nurse-event` (the Tasks runtime already subscribes), then cross-reference `~/.hyvemind/debug/nurse/decisions.jsonl.*` for the decision chain.

### "Nurse didn't intervene on a session that was clearly stuck"

```bash
# 1. Find the session_id (from progress_log.jsonl or the UI).
SID=sess-abc123

# 2. List every decision that touched this session in the last 24h.
DATE=$(date +%Y-%m-%d); YESTERDAY=$(date -v-1d +%Y-%m-%d 2>/dev/null || date -d 'yesterday' +%Y-%m-%d)
jq -c "select(.session_id == \"$SID\")" \
  ~/.hyvemind/debug/nurse/decisions.jsonl.$DATE \
  ~/.hyvemind/debug/nurse/decisions.jsonl.$YESTERDAY 2>/dev/null | \
  jq -s 'group_by(.decision_id) | map({decision_id: .[0].decision_id, events: map({event, data})})'

# 3. Look for the gated outcomes. The decision_finalised row tells you why nothing happened.
jq -c "select(.session_id == \"$SID\" and .event == \"decision_finalised\")" \
  ~/.hyvemind/debug/nurse/decisions.jsonl.$DATE | jq '.data.status'

# 4. If status was "gated_budget", check the budget_evaluated rows to see which detector class was exhausted.
# 5. If status was "gated_storm_guard", check the storm_guard_evaluated rows.
# 6. If status was "gated_post_lag", cross-reference bus.jsonl for the lag event.
# 7. If status was "classifier_skipped_no_model", check ProfileConfig.nurse_model
#    (per-profile override) and NurseConfig.nurse_model (engine-wide fallback) in config.json.
# 8. If status was "fast_path_awaiting_model" or "fast_path_healthy_streaming", the
#    dispatcher's Watchdog fast-path concluded the session was making progress and
#    short-circuited the tier pipeline — the corresponding tier1_evaluated row will say so.
# 9. If status was "dispatched_synthesized", a non-session signal (e.g. Hivemind context-gather
#    error) ran the synthesized path — see §D.7 / the nurse README synthesized chain note.
# 10. If you don't see ANY decisions for the session_id, look at signals/{SID}.jsonl —
#     detectors never raised. Cross-reference the per-session debug log to see what the
#     agent session was doing.
```

### "Nurse intervened, but with the wrong action"

```bash
DID=8a3c9f0e2b6c4d5a  # decision_id from the UI or from the Intervention Log
jq -c "select(.decision_id == \"$DID\")" ~/.hyvemind/debug/nurse/decisions.jsonl.* | \
  jq -s 'sort_by(.event_seq)'

# Pull the Tier 3 capture if one fired
cat ~/.hyvemind/debug/nurse/captures/${DID}-prompt.txt
cat ~/.hyvemind/debug/nurse/captures/${DID}-response.txt
```

### "Nurse intervened but the session didn't recover"

```bash
DID=...
# The intervention_outcome rows tell you. There are two of them per decision (t=0 and t+5m).
jq -c "select(.decision_id == \"$DID\" and .event == \"intervention_outcome\")" \
  ~/.hyvemind/debug/nurse/decisions.jsonl.* | jq '.data'

# If label == "intervention_ineffective", look at signals_active_at_+5m to see which
# detector keys persisted past the intervention. That's where to focus the next fix.
```

### "Kill verification failed"

```bash
DID=...
jq -c "select(.decision_id == \"$DID\" and .event == \"kill_verification\")" \
  ~/.hyvemind/debug/nurse/decisions.jsonl.* | jq '.data'

# The full timeline (abort_sent_at → liveness_check_at_t3s → still_alive → force_kill_sent_at → dead_at)
# tells you whether the session ignored the abort, ignored the force_kill, or eventually died.
# If result == "double_fail_giving_up", the safety circuit terminated retry — the session
# is leaked but the budget is no longer being eaten by repeated Cancel attempts.
```

### "Nurse is laggy / dropping events"

```bash
# bus.jsonl shows every lag event and capacity-pressure breach.
jq -c "select(.kind == \"lag\" or .kind == \"capacity_pressure\")" \
  ~/.hyvemind/debug/nurse/bus.jsonl.* | jq '.data'

# If lag is frequent, bump HYVEMIND_NURSE_BUS_CAPACITY (clamped 64-65,536) or
# diagnose what's blocking the engine loop (often a slow classifier call;
# the slow-probe-task split is meant to prevent this).
```

### "Is the Tier 3 classifier actually hitting the provider prompt cache?"

The Nurse classifier opts into provider-side prompt caching (`CallRequest::with_cache_static_prefix(true)` in `nurse/classifier.rs`). Anthropic honours this via `cache_control: ephemeral` markers; DeepSeek (and every OpenAI-compatible backend) ignores the flag and caches automatically when the request prefix is byte-stable. Every Tier 3 call's cache savings land on the `classifier_returned` event:

```bash
# Per-call cache hit/write tokens and derived hit ratio.
jq -c 'select(.event == "classifier_returned") | .data
  | { decision, provider, model,
      input: .input_tokens, output: .output_tokens,
      cache_hit: .cache_hit_tokens, cache_write: .cache_write_tokens,
      hit_ratio: .cache_hit_ratio }' \
  ~/.hyvemind/debug/nurse/decisions.jsonl.$(date +%Y-%m-%d) | tail -10
```

What to expect:
- **DeepSeek (`provider: "deepseek"`):** `cache_hit_tokens` should be ≈ 6,000+ on the second and subsequent calls within ~5 minutes of the same prefix shape. First call: 0. `cache_write_tokens` stays 0 (DeepSeek has no equivalent metric).
- **Anthropic (`provider: "anthropic"`):** first call shows `cache_write_tokens ≈ 6,000`; subsequent calls show `cache_hit_tokens ≈ 6,000`, `cache_write_tokens` 0.
- **All zero on every call:** the prefix is unstable. Prime suspect is the `OpenAICompatibleProvider` `tool_choice` fallback — once a model is recorded in `auto_tool_choice_models`, every subsequent call uses the identical `"auto"` byte string (`providers/mod.rs:373-377`); only the very first call to a freshly-seen problematic model is a miss.

### IPC mirrors of these recipes

The same data is reachable from the frontend (and from a Bash tool call against `tauri` IPC) via:

| Command | Purpose |
|---|---|
| `get_nurse_decision_chain(decision_id)` | Full ordered event list for one decision (reads `decisions.jsonl.*`). |
| `get_nurse_decisions_for_session(session_id, since_ts?, limit?)` | All decision_ids that ran on a session. |
| `get_nurse_signal_stream(session_id, since_ts?, limit?)` | Per-session signal raise/clear stream (wraps `signals/{session_id}.jsonl`). |
| `get_nurse_capture(decision_id, kind)` | Stream a Tier-3 capture file (256 KB read cap). |
| `export_nurse_diagnostic_bundle(decision_id?)` | Zip the decision chain + captures + ±5min signal stream for the affected session and return the path — designed for "drop this into Claude Code". |

## Investigating a Swarm — `progress_log.jsonl`

Each swarm writes an append-only JSONL log at `~/.hyvemind/swarms/{swarm_id}/progress_log.jsonl`. The first line is a `{"schema_version": 2}` header; logs that lack it are treated as `schema_version=1`. The replay reader (`ProgressReader::rebuild_state` and `rebuild_session_associations` in `state/progress.rs`) skips the header and tolerates a truncated tail.

### Progress event types (`ProgressEvent.event_type`)

| Event | When | Key Data |
|-------|------|----------|
| `swarm_started` | Queen begins execution | swarm-level only |
| `swarm_completed` / `swarm_failed` / `swarm_paused` / `swarm_resumed` | Swarm lifecycle transitions | swarm-level only |
| `feature_started` | Scout or Worker phase begins on a feature | `feature_id`, `run_id` |
| `feature_scouted` | Scout produced a plan | `feature_id`, `run_id` |
| `feature_implemented` | Worker phase completed (handoff parsed) | `feature_id`, `run_id` |
| `feature_validated` | Guard passed / feature reached `Completed` | `feature_id`, `run_id` |
| `feature_failed` | Feature terminal-failed | `feature_id`, `run_id` |
| `feature_skipped` | Cancelled or dependency unmet | `feature_id` |
| `nurse_intervention` | Nurse Steer/Restart/Diagnose | metadata.intervention_kind |
| `guard_validation` | Guard run started / produced result | `feature_id`, `run_id` (where known) |
| `hivemind_review_started/_completed/_skipped` | Optional Hivemind review on scout/queen plan | `feature_id`, metadata.hivemind_id |
| `discovered_issue` | Worker surfaced a non-blocking issue | `feature_id`, metadata.severity/description |
| `budget_exceeded` | Per-swarm or daily cap hit; queen pauses next batch | metadata.reason/scope/spend/cap |
| `pi_session_spawned` | An agent session was spawned for a swarm role (event name kept for log back-compat) | metadata.session_id/role/feature_id/pid |
| `pi_session_killed` | A previously-spawned agent session was killed (event name kept for log back-compat) | metadata.session_id/reason |
| `worker_handoff` | Worker emitted its structured handoff JSON | `feature_id`, `run_id`, metadata.success_state |
| `guard_attempt` | Guard started a new validation attempt | `feature_id`, `run_id` (where known), metadata.attempt/milestone_id |
| `error` | Generic error event | metadata may carry details |

Every event carries an optional `run_id` so retries on the same feature are distinguishable. The Queen mints one `run_id` per `run_feature_full` invocation; Scout / Worker / inline Guard / validator Guard for the same attempt all share it.

### Commands

```bash
# Pretty-print the full progress log
cat ~/.hyvemind/swarms/{SWARM_ID}/progress_log.jsonl | python3 -m json.tool --no-ensure-ascii

# Detect schema version (header line on row 1)
head -1 ~/.hyvemind/swarms/{SWARM_ID}/progress_log.jsonl | python3 -m json.tool

# Filter to just one feature's run history (latest run_id wins)
python3 -c "
import json, sys
target = sys.argv[2]
for line in open(sys.argv[1]):
    try: e = json.loads(line)
    except: continue
    if e.get('feature_id') == target:
        ts = e.get('timestamp','')[:19]
        rid = e.get('run_id','-')
        et = e.get('event_type','')
        print(f'{ts} {rid} {et} {e.get(\"message\",\"\")[:80]}')
" ~/.hyvemind/swarms/{SWARM_ID}/progress_log.jsonl {FEATURE_ID}

# Find agent sessions that were spawned but never killed (crash victims)
python3 -c "
import json, sys
spawned, killed = {}, set()
for line in open(sys.argv[1]):
    try: e = json.loads(line)
    except: continue
    md = e.get('metadata') or {}
    if e.get('event_type') == 'pi_session_spawned':
        spawned[md.get('session_id','?')] = (e.get('feature_id'), md.get('pid'), md.get('role'))
    elif e.get('event_type') == 'pi_session_killed':
        killed.add(md.get('session_id','?'))
orphan = [(s,*v) for s,v in spawned.items() if s not in killed]
for s,fid,pid,role in orphan: print(f'orphan session={s} role={role} pid={pid} feature={fid}')
" ~/.hyvemind/swarms/{SWARM_ID}/progress_log.jsonl
```

### Sibling log: `activity_log.jsonl`

Alongside `progress_log.jsonl` each swarm also writes `~/.hyvemind/swarms/{swarm_id}/activity_log.jsonl` — the firehose feed consumed by the `swarm-activity` Tauri event after the 50ms/256-byte text/thinking coalescer in `commands/swarms.rs`. Every line is one IPC payload (`SwarmActivityEvent` in `app/src/lib/events.ts`) augmented with a monotonic per-swarm `seq` field. Schema-versioned via a `{"schema_version": 1}` header on the first line; truncated tails are tolerated. The frontend pages through it via `get_swarm_activity_log(swarm_id, after_seq, limit)` on mount so SwarmControl can replay history even if the user opens the panel after an agent has already streamed events.

```bash
# Pretty-print the full activity log
cat ~/.hyvemind/swarms/{SWARM_ID}/activity_log.jsonl | python3 -m json.tool --no-ensure-ascii
```

## Tauri Commands (IPC)

All registered in `lib.rs` via `invoke_handler!`. Every command returns `Result<T, IpcError>` (see `state/ipc_error.rs` + `app/src/lib/ipc.ts`). The lib.rs registration is the **authoritative list** — when adding or removing one, update this section.

The exhaustive per-command list and per-handler signatures / return types / delegation targets live in **`docs/ipc-reference.md`** (authoritative source: the `invoke_handler!` registration in `lib.rs`) — the command names drift, so they're not enumerated here. Approximate bucket grouping for navigation (~119 production + 1 debug-only):

| Bucket | Module | ~Count |
|--------|--------|-------:|
| Tasks (internal: chat) | `commands/chat.rs` | 7 |
| Tasks (UI state) | `commands/tasks.rs` | 6 |
| Hivemind | `commands/hivemind.rs` | 19 + 1 dbg |
| Swarms | `commands/swarms.rs` | 15 |
| Settings | `commands/settings.rs` | 33 |
| Dashboard | `commands/dashboard.rs` | 5 |
| Extensions | `commands/extensions.rs` | 4 |
| Nurse | `commands/nurse.rs` | 18 |
| Sessions | `commands/sessions.rs` | 5 |
| Tests | `commands/tests.rs` | 7 |

**Chat-history semantics post-OpenCode**: `get_chat_history` rebuilds from the live `SessionRuntime` in-memory transcript only (no on-disk transcripts exist); `list_chat_sessions` always returns an empty list (kept for IPC compat); `delete_chat_session` closes the live session (no file deletion); `get_session_last_assistant_text` is best-effort from the live runtime. The frontend's own per-task message store (`commands/tasks.rs`, `~/.hyvemind/task-messages/`) is the durable conversation history.

### Choosing the right `IpcError` variant

When raising an error from a command, pick the variant the frontend's `formatIpcError` and per-kind branching expect. `IpcError::from_provider_error` (`state/ipc_error.rs:205`) auto-classifies stringy provider errors by substring (`401` / `429` / `circuit breaker open`).

| Use this | When |
|---|---|
| `validation` | User-attributable: empty id, path traversal, oversized payload, malformed body, illegal model name |
| `not_found { resource, resource_id }` | A swarm/session/review/hivemind/extension lookup missed (renamed fields avoid colliding with the discriminator `kind`) |
| `not_approved` | Working-directory allowlist rejected the path, or subscription auth missing |
| `provider_unauthenticated` / `provider_rate_limited` / `circuit_breaker_open` | Lift from provider error string via `IpcError::from_provider_error` |
| `internal` | I/O error, logic bug, panic, upstream 5xx — `details.chain` carries the full `anyhow` chain |

Free-form `String` errors lift into `IpcError::Internal` via the blanket `From<String>`. Avoid relying on it — it loses the discriminator and breaks the frontend's branched handlers (e.g., `not_approved` → approval modal).

**Session-ID validator note.** Composite swarm session ids (`{role}-{swarm_uuid}-{feature_id}` minted in `core/queen.rs::run_feature_full`) can exceed 64 chars with realistic LLM-generated feature slugs. IPCs that take a `session_id` MUST use `validate_session_id` (128-char cap, `commands/util.rs`), not the shared 64-char `validate_id` — swapping this back caused the SwarmControl bottom bar to render `↑0 / ↓0 / 0%` for the lifetime of a Worker. Every other ID (`swarm_id`, `review_id`, `task_id`, `hivemind_id`, `feature_id`, …) stays on `validate_id` because their lengths feed filesystem-path budgets under `~/.hyvemind/`.

### Hivemind command gotchas

- `start_review`: the `stance` argument is **silently ignored** — hardcoded `Against`. `For` / `Neutral` exist in the schema but are not wired (PRODUCT.md §6).
- `start_review`: `round_number` is the 1-based cumulative round for the Tasks-flow driver; collapses to `round_offset = round_number - 1` so capture files (`merge-rN.txt`, `output-*-rN.txt`) don't overwrite earlier rounds. Preserve this offset when adding round-related fields.
- `delete_review`: refuses if any child job is `pending` / `running` / `round_*` — returns `validation` error `"Cannot delete a running review. Cancel it first."`. Call `cancel_review` first.
- `register_context_session` MUST be called when the orchestrator spawns a context-gather agent session, or `get_orchestrator_usage` cannot attribute its tokens to the review.
- `clear_response_cache` is **debug-builds-only** (`#[cfg(debug_assertions)]` in `lib.rs`); never depend on it in production code paths.

## Tauri Events (backend → frontend)

Backend code emits via `app_handle.emit("…", payload)`; the frontend subscribes via `listen("…", cb)` (see §Frontend Architecture → Event listener index for the consumer side).

| Event | Emitted by | Payload purpose |
|-------|-----------|-----------------|
| `chat-event` | `commands/chat.rs` (via the 100ms / 50-token IPC coalescer in the chat command) | Streaming Tasks-view tokens + lifecycle (start / chunk / done / error) |
| `hivemind-progress` | `hivemind/engine.rs`, `hivemind/review_log.rs`, `lib.rs` startup sweep | Review round progress, model completions, lifecycle events (incl. `merge_interrupted` / `review_interrupted` from crash recovery) |
| `swarm-event` | `core/queen.rs`, `nurse/intervention.rs` (`DefaultApplier`) | Swarm lifecycle (feature status changes, Nurse interventions, completion/failure) |
| `swarm-activity` | `commands/swarms.rs` (after the 50ms/256-byte coalescer) | Per-feature agent activity stream (harness events: text/thinking deltas, tool calls) |
| `swarm_reconciled` | `lib.rs` setup hook | Fan-out for swarms the progress-log replay flagged as `Interrupted` — frontend uses it to render Resume badges immediately |
| `nurse-event` | `nurse/intervention.rs` (`DefaultApplier` + `dispatch_synthesized`), `commands/nurse.rs` (`set_nurse_config` StatusUpdate) | Nurse status updates + intervention notifications (also mirrored from `swarm-event` for Tasks-view consumption). Wire shape — `NurseLifecyclePayload` / `NurseStatusSnapshot` / `NurseEvent` — lives in `nurse/snapshot.rs`. |
| `test-progress` | `core/stability_test/runner.rs` | Stability-test phase, status, progress updates |
| `usage-snapshot-updated` | `extensions/poller.rs` | Provider usage/balance snapshots pushed by extension pollers |
| `default-model-changed` | `commands/settings.rs::set_default_model` | Default-model selection changed in Settings |
| `default-project-path-changed` | `commands/settings.rs::set_default_project_path` | Default project-path changed in Settings |
| `default-hivemind-changed` | `commands/settings.rs::set_default_hivemind` | Default Hivemind selection changed in Settings |
| `session-evicted` | `harness/eviction.rs` (maintenance loop) | An idle session was evicted; a synthetic `chat-event` notice is emitted alongside it |

## Environment Variables

### Credentials and logging

- `ANTHROPIC_API_KEY` — Anthropic API key (loaded at startup, overrides config file)
- `OPENAI_API_KEY` — OpenAI API key
- `OPENROUTER_API_KEY` — OpenRouter API key
- `HYVEMIND_DEBUG=1` — Enable debug mode (TRACE-level structured JSON logs to disk)
- `RUST_LOG` — Override stderr log level (e.g. `RUST_LOG=debug`)
- `HYVEMIND_OPENCODE_BIN` — Absolute path override for the `opencode` binary (otherwise PATH → `/opt/homebrew/bin` → `/usr/local/bin`). Sessions are spawned on demand by `send_message` against the shared `opencode serve` process; the 10-min idle eviction reclaims them after a user goes quiet.

### Tunables (defined in `src/tunables.rs`)

All values are read on every accessor call, so overrides take effect at any
process start without rebuilding. Invalid / unparseable values silently fall
back to the default.

- `HYVEMIND_CONCURRENCY_CAP` — max concurrent model calls per Hivemind round (default `8`)
- `HYVEMIND_ROUND_TIMEOUT_SECS` — fallback per-round timeout when a `RoundConfig` doesn't supply one (default `450`)
- `HYVEMIND_DEFAULT_MAX_TOKENS` — default `max_tokens` for providers that need one (currently Anthropic; default `4096`)
- `HYVEMIND_RESPONSE_CACHE_SIZE` — max entries in the per-review response cache (default `1000`)
- `HYVEMIND_RESPONSE_CACHE_TTL_SECS` — per-entry TTL for the response cache (default `3600`)
- `HYVEMIND_SWARM_FEATURE_PARALLELISM` — upper clamp on a swarm's `max_concurrent_features` (default `6`)
- `HYVEMIND_DEBUG_LOG_RETENTION_DAYS` — debug log retention window in days (default `7`)
- `HYVEMIND_LOG_CHANNEL_CAPACITY` — bounded capacity for the async tracing log channel (default `4096`)
- `HYVEMIND_CIRCUIT_BREAKER_THRESHOLD` — consecutive failures before the per-provider circuit breaker opens (default `5`)
- `HYVEMIND_CIRCUIT_BREAKER_COOLDOWN_SECS` — Open → HalfOpen cooldown for the breaker (default `60`)
- `HYVEMIND_PROVIDER_TIMEOUT_SECS` — default HTTP request timeout for Anthropic / OpenAI-compatible providers (default `120`)
- `HYVEMIND_INACTIVITY_TIMEOUT_SECS` — per-turn recv inactivity timeout for the collect loop (default `1800`, clamp `[60, 7200]`); sized for long-running tool executions and silent server-side reasoning
- `HYVEMIND_OPENCODE_STREAM_SILENCE_TIMEOUT_SECS` — streaming-silence timeout, applied while no tool execution is in flight (default `180`, clamp `[30, 1800]`). OpenCode streams reasoning as deltas, so a silent mid-message stream means a dead provider stream; while a tool runs, the long inactivity timeout applies instead.
- `HYVEMIND_OPENCODE_BIN` — absolute path override for the user-installed `opencode` binary (see §OpenCode runtime)

#### Nurse tunables

All clamps applied silently — out-of-range values fall back to the default.

- `HYVEMIND_NURSE_BUS_CAPACITY` — broadcast capacity for `NurseBus` (default `4096`, clamp `[64, 65536]`). Lag events fire on `RecvError::Lagged(n)`; sustained lag triggers post-lag Tier 2/3 suppression for affected sessions.
- `HYVEMIND_NURSE_MAX_EVIDENCE_BYTES` — per-`Signal` evidence cap (default `8192`, clamp `[1024, 65536]`).
- `HYVEMIND_NURSE_STALL_THRESHOLD_SECS` — `StallDetector` default threshold (default `180`, clamp `[60, 3600]`); per-profile overrides in `ProfileConfig`.
- `HYVEMIND_NURSE_TICK_INTERVAL_SECS` — engine tick cadence (default `10`, clamp `[5, 600]`).
- `HYVEMIND_NURSE_PROVIDER_TIMEOUT_SECS` — Tier 3 classifier provider-call timeout (default `90`, clamp `[10, 600]`).
- `HYVEMIND_NURSE_SLOW_PROBE_INTERVAL_SECS` — interval for the slow-tick task that runs `Detector::tick_kind() == Slow` detectors (default `10`, clamp `[5, 600]`).
- `HYVEMIND_NURSE_DECISION_LOG_RETENTION_DAYS` — `decisions.jsonl.*` + capture-file mtime retention (default `30`, clamp `[1, 365]`).
- `HYVEMIND_NURSE_SIGNAL_STREAM_MAX_BYTES` — `signals/{session_id}.jsonl` rotation threshold (default `4 MiB`, clamp `[64 KiB, 64 MiB]`).
- `HYVEMIND_NURSE_BUS_LOG_RETENTION_DAYS` — `bus.jsonl.*` mtime retention (default `30`, clamp `[1, 365]`).
- `HYVEMIND_NURSE_CAPTURE_MAX_BYTES` — per-capture (`prompt.txt` / `response.txt`) size cap (default `1 MiB`, clamp `[16 KiB, 16 MiB]`).
- `HYVEMIND_NURSE_OBSERVABILITY_QUEUE_DEPTH` — bounded mpsc capacity for each observability writer (default `2048`, clamp `[128, 65536]`). Overflow drops the event silently and increments a counter exposed via `get_nurse_engine_status`.
- `HYVEMIND_NURSE_BATCH_INTERVAL_SECS` — cadence of the `BatchReviewer` batched LLM review tick (default `120`, clamp `[30, 3600]`).
- `HYVEMIND_NURSE_BATCH_EVENTS_PER_SESSION` — max recent `HarnessEvent`s pulled per session for a batched-review prompt (default `200`, clamp `[20, 1000]`).
- `HYVEMIND_NURSE_BATCH_CHARS_PER_SESSION` — per-session character cap for the rendered transcript block (default `3000`, clamp `[500, 30000]`); the dominant lever for batch-prompt size.

### Nurse internal constants (not env-tunable)

Compile-time tuning knobs across the `nurse/` module. Not user-tunable via `HYVEMIND_*` env vars — change in code only.

- `SELF_KILL_GRACE = 30s` (`nurse/dispatcher.rs`) — after a `Restart` / `Cancel` self-kill, suppresses any further dispatch on the same session for 30s. Prevents re-entrant Restart loops while the new session boots.
- `kill_with_verification` grace windows (`nurse/intervention.rs`) — abort → 3s grace poll → `kill_session` → 7s post-kill poll → `dead_at` row OR `double_fail_giving_up` (no retry).
- Storm-guard window (`nurse/storm_guard.rs`) — per-(session_id, dedup_key) sliding window; default 3 events / 60s before gating. Tier 1 bypasses storm guard.
- LLM classifier provider-call timeout — 90s, governed by `HYVEMIND_NURSE_PROVIDER_TIMEOUT_SECS` (clamped `[10, 600]`). The classifier-end of the dispatcher pipeline.
- Watchdog respawn budget — see `super_watchdog` in §Key Types.

## Debug Mode (checking logs)

When the user says "check logs", this means examine the debug log files for errors, warnings, or unexpected behavior.

### Enabling debug mode

```bash
HYVEMIND_DEBUG=1 cargo run          # from app/src-tauri
```

### Log locations

Debug logs are routed per-ID by `state/log_routing.rs`. Events fire inside `#[tracing::instrument]` spans that carry an ID field, and the layer writes each event to the matching bucket:

| Path | Contains |
|------|----------|
| `~/.hyvemind/debug/sessions/{session_id}.jsonl` | All TRACE/DEBUG/INFO/WARN/ERROR for one Tasks-view session (harness events, chat command, session lifecycle) |
| `~/.hyvemind/debug/reviews/{review_id}.jsonl` | All events for one Hivemind review (engine, providers, merge) |
| `~/.hyvemind/debug/swarms/{swarm_id}/swarm.jsonl` | Swarm orchestration (Queen loop, scheduler, top-level Nurse) |
| `~/.hyvemind/debug/swarms/{swarm_id}/{agent}-{feature_id}-{run_id}.jsonl` | One agent run on one feature (e.g. `worker-feat-001-run-7.jsonl`); missing pieces collapse to `worker.jsonl` / `worker-feat-001.jsonl` |
| `~/.hyvemind/debug/general.jsonl.YYYY-MM-DD` | Startup, library logs, anything that fires outside a known span (daily rotation) |

Routing priority (most specific wins): `review_id` > `swarm_id`+`agent` > `swarm_id` > `session_id` > `general`. Files older than 7 days are pruned on startup. Stderr stays at INFO. **No debug files exist unless `HYVEMIND_DEBUG=1` was set when the event fired.**

#### Nurse always-on observability

`~/.hyvemind/debug/nurse/` is the exception — it is written **regardless of `HYVEMIND_DEBUG`**. The reasoning: in alpha we cannot reproduce a 4-hour swarm failure on demand. Nurse misfires (wrong action, no fire, kill-verification failed) must be diagnosable from a single user-submitted bundle without asking them to turn on debug and try again.

| Path | Contains | Retention |
|------|----------|-----------|
| `~/.hyvemind/debug/nurse/decisions.jsonl.YYYY-MM-DD` | Full per-decision event chain — `decision_started` / `storm_guard_evaluated` / `tier1_evaluated` / `playbook_evaluated` / `classifier_invoked` / `classifier_returned` / `intervention_dispatched` / `kill_verification` / `intervention_outcome` / `decision_finalised`. Every row carries `decision_id`, `session_id`, `event_seq` (monotonic per decision). | `HYVEMIND_NURSE_DECISION_LOG_RETENTION_DAYS` (default 30 days) |
| `~/.hyvemind/debug/nurse/captures/{decision_id}-prompt.txt` and `{decision_id}-response.txt` | Verbatim Tier 3 classifier prompt + response, plain-text. Prompt is written BEFORE the provider call so a crash leaves an unambiguous "invoked, never returned" trace. Capped at `HYVEMIND_NURSE_CAPTURE_MAX_BYTES` (default 1 MiB). | Pruned alongside `decisions.jsonl` |
| `~/.hyvemind/debug/nurse/signals/{session_id}.jsonl` | Every `SignalDelta::Raise` / `::Clear` for that session, with full `evidence` payload. Rotated `.1` / `.2` / `.3` at `HYVEMIND_NURSE_SIGNAL_STREAM_MAX_BYTES` (default 4 MiB), then drops. | Pruned 24h after `SessionEnded` |
| `~/.hyvemind/debug/nurse/bus.jsonl.YYYY-MM-DD` | Bus-level lifecycle — `SessionSpawned` / `SessionEnded` / `OwnerChanged` / `lag` / `capacity_pressure` (sampled 1/s when broadcast crosses 80%) / `post_lag_suppression_entered`/`_exited` / `dropped_for_unknown_session`. | `HYVEMIND_NURSE_BUS_LOG_RETENTION_DAYS` (default 30 days) |

Always-on Nurse footprint is bounded: ≤200 KB/day of `decisions.jsonl`, ≤500 KB/day of `bus.jsonl`, plus per-session signal streams (4 MiB hard cap each) and per-decision capture files. A typical week stays under ~50 MB.

The dispatcher writes to `decisions.jsonl.*` as the single source of truth for what fired and why. The `nurse-event` IPC + `progress_log.jsonl` `nurse_intervention` rows record the user-visible action.

### How to check logs

```bash
# Inspect a specific session end-to-end
cat ~/.hyvemind/debug/sessions/{session_id}.jsonl | python3 -m json.tool

# Inspect a specific hivemind review end-to-end
cat ~/.hyvemind/debug/reviews/{review_id}.jsonl | python3 -m json.tool

# Inspect one swarm's full activity
ls ~/.hyvemind/debug/swarms/{swarm_id}/
cat ~/.hyvemind/debug/swarms/{swarm_id}/*.jsonl | python3 -m json.tool

# Pull just one Worker's transcript for one feature
cat ~/.hyvemind/debug/swarms/{swarm_id}/worker-{feature_id}-{run_id}.jsonl | python3 -m json.tool

# General / uncategorized events (startup, library messages)
tail -20 ~/.hyvemind/debug/general.jsonl.$(date +%Y-%m-%d) | python3 -m json.tool

# Errors across all sessions today
grep -r '"level":"ERROR"' ~/.hyvemind/debug/sessions/ ~/.hyvemind/debug/reviews/ ~/.hyvemind/debug/swarms/ | python3 -m json.tool

# OpenCode SSE / request traffic for a specific session (truncated previews at TRACE)
grep -i 'sse\|opencode' ~/.hyvemind/debug/sessions/{session_id}.jsonl | python3 -m json.tool

# LLM provider request/response payloads for a specific review
grep 'provider request\|provider response\|anthropic request\|anthropic response' ~/.hyvemind/debug/reviews/{review_id}.jsonl | python3 -m json.tool

# Nurse interventions on a specific swarm
grep 'nurse' ~/.hyvemind/debug/swarms/{swarm_id}/*.jsonl | python3 -m json.tool

# Circuit breaker state changes (usually in a review or session bucket)
grep -r 'circuit' ~/.hyvemind/debug/ | python3 -m json.tool
```

### What's logged at each level

| Level | What | Where |
|-------|------|-------|
| `ERROR` | Failures (crashed processes, failed API calls, panics) | stderr + debug file |
| `WARN` | Stall detections, circuit breaker trips, timeouts, retries | stderr + debug file |
| `INFO` | Command invocations, session lifecycle, high-level status | stderr + debug file |
| `DEBUG` | Prompt/response summaries (lengths, previews), scheduler state, session counts, config details | debug file only |
| `TRACE` | Raw OpenCode SSE lines (truncated previews + length), full HTTP request bodies, atomic writes, circuit breaker internal state | debug file only |

Full LLM responses are NOT inlined in the JSONL. For hivemind reviews they live in per-call capture files (`~/.hyvemind/reviews/{review_id}/output-{model}-r{round}.txt`); for Tasks-view sessions the conversation lives server-side in OpenCode (the frontend's durable copy is `~/.hyvemind/task-messages/`).

### Log rotation

- Per-ID files (`sessions/`, `reviews/`, `swarms/`) accumulate as long as their entity exists; pruned by mtime after 7 days on startup
- `general.jsonl.YYYY-MM-DD` rotates daily and is pruned by date after 7 days
- Writes are non-blocking: events queue into a bounded `mpsc::sync_channel(4096)` consumed by a worker thread that owns up to 64 open file handles (LRU eviction)
- Channel overflow drops the event silently — the runtime never stalls on log I/O

### rtk proxy warning

If using `rtk` (Rust Token Killer), its hook rewrites shell commands and **may suppress or filter output** from `ls`, `cat`, `tail`, `grep`, etc. When investigating log files, bypass rtk:

```bash
rtk proxy ls -la ~/.hyvemind/debug/        # explicit proxy passthrough
rtk proxy tail -50 ~/.hyvemind/debug/...   # or use rtk proxy prefix
```

### Frontend devtools

Open the Tauri webview console: **Cmd + Option + I** (macOS) or right-click → "Inspect Element". The console shows:
- Frontend IPC calls and errors
- Tauri event listener registrations
- React rendering issues
- Network requests (if any from the webview)

### Debugging a stuck Tasks-view session

When a Task spinner hangs without producing output:

1. **Check backend logs** — read the per-session file directly (no grep needed; everything in there is for that one session):
   ```bash
   rtk proxy tail -200 ~/.hyvemind/debug/sessions/{SESSION_ID}.jsonl | python3 -c "
   import json, sys
   for line in sys.stdin:
       try:
           obj = json.loads(line.strip())
           ts = obj.get('timestamp','')[:19]
           lvl = obj.get('level','')
           target = obj.get('target','')
           fields = obj.get('fields',{})
           msg = fields.get('message','')
           extra = ''
           for k in ('event_type','type'):
               if k in fields: extra += f' {k}={fields[k]}'
           print(f'{ts} [{lvl}] {target}: {msg}{extra}')
       except: pass
   "
   ```

2. **Interpret the results**:
   - SSE events flowing (text/thinking deltas being parsed) → the OpenCode session is alive, the model is responding (may just be slow)
   - Stream-silence timeout (`HYVEMIND_OPENCODE_STREAM_SILENCE_TIMEOUT_SECS`, default 180s) fired mid-message → the provider stream died; the turn errors and a best-effort abort is sent
   - `opencode serve` exited / `ProcessCrashed` → the shared server died; every session channel gets a fail-fast error and the harness respawns the server on the next spawn
   - No events after `sending prompt` → model API is unresponsive or network issue

3. **Check the Nurse surfaces** (always on): `~/.hyvemind/debug/nurse/signals/{SESSION_ID}.jsonl` shows whether detectors saw the stall; the decision log shows what (if anything) Nurse did about it.

4. **Common causes of stuck sessions**:
   - **Slow model** — some models (especially large ones or free-tier endpoints) can take 30-120s
   - **Rate limiting** — look for auto-retry events; the session is waiting to retry
   - **API key issue** — check Settings page; the provider should show "Configured" with a green dot
   - **OpenCode server crash** — look for ERROR level logs mentioning `opencode serve` / `ProcessCrashed`
   - **Frontend event listener missed** — open devtools console, check for JS errors

5. **Distinguish backend vs frontend issues**:
   - If backend logs show events flowing but the UI is stuck → frontend issue (check devtools console)
   - If backend logs stop after `sending prompt` → model/API issue
   - If backend logs show errors → backend issue

### Debugging a hivemind review

The hivemind review engine dispatches model calls through the `ProviderRegistry`. When a review returns stub/placeholder data or fails:

1. **Check the review log** (requires `HYVEMIND_DEBUG=1`):
   ```bash
   # High-level events
   cat ~/.hyvemind/reviews/{REVIEW_ID}.jsonl | python3 -m json.tool
   # Full TRACE firehose for the same review
   cat ~/.hyvemind/debug/reviews/{REVIEW_ID}.jsonl | python3 -m json.tool
   # A specific model's full output
   cat ~/.hyvemind/reviews/{REVIEW_ID}/output-{model_id_safe}-r{round}.txt
   ```

2. **Common issues**:
   - `provider 'X' not found in registry` → API key not configured for that provider
   - `duration_ms: 0` with placeholder text → stub response (provider not wired up)
   - `circuit breaker open` → too many consecutive failures for that provider
   - All models show identical token counts → responses are cached or stubbed

3. **Verify the provider registry loaded correctly** — startup logs land in the daily general bucket:
   ```bash
   grep 'provider registry refreshed' ~/.hyvemind/debug/general.jsonl.$(date +%Y-%m-%d) | python3 -m json.tool
   ```

---
> Source: [Unravl/Hyvemind](https://github.com/Unravl/Hyvemind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
