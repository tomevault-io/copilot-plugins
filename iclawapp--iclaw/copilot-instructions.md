## iclaw

> Hi. If you're an AI coding agent (Claude Code, Codex, Cursor, anyone) about to make changes here, read this once before you touch the source.

# AGENTS.md — Guardrails for AI coding agents working on iClaw

Hi. If you're an AI coding agent (Claude Code, Codex, Cursor, anyone) about to make changes here, read this once before you touch the source.

## What iClaw is

A **local web UI for [OpenClaw Gateway](https://docs.openclaw.ai)**. Nothing more. The app:

- Renders a ChatGPT-style chat surface (sidebar of chats, conversation in the middle).
- Stores chat history locally in SQLite (`data/iclaw.db`).
- Talks to a running OpenClaw Gateway on the same machine via the **native WebSocket protocol** (no HTTP chat completions, no SSE).
- Auto-loads the bearer token from `~/.openclaw/openclaw.json`.
- Adds higher-level features that the gateway doesn't have natively: per-project shared facts, scheduled messages, slash autocomplete, header model picker, today's cost chip, exec approval cards.

## What iClaw is **not**

These are intentional non-goals. Don't add them. If a request lands here that needs any of them, push back or open an issue first.

- **Not a generic OpenAI-compat client.** We migrated off `/v1/chat/completions` in commit `a92c10e`. Do not re-introduce an HTTP chat completion path or SSE-based streaming — everything chat-related is native WS.
- **Not a multi-provider chat tool.** No fallback to Ollama, vLLM, Claude API, OpenAI direct, etc. OpenClaw is the only backend; switch model providers in `openclaw.json`.
- **Not a remote/hosted product.** Loopback-only. No multi-user, no auth layer, no team features. Threat model is "this user on this machine".
- **For the default Full Power chat, not a re-implementation of OpenClaw's agent runtime.** On that path we don't compact transcripts, run tools, or wrap the LLM — we display what the gateway produces and (separately) maintain a project-scoped fact layer *additional* to OpenClaw memory. (The Work / Safe work / Incognito modes are the deliberate exception — a small, self-contained agent loop in `packages/iclaw-runtime`; see below. Don't grow it into a general OpenClaw replacement.)
- **Not heavyweight on the frontend.** No build step beyond `tsc`. Plain CSS, vanilla JS on the client, EJS for views. If you want to add React/Vue/Svelte, open an issue first — the answer is probably no for the current scope.

## Architecture you need to know

```
                  ┌─────────────────┐
                  │  OpenClaw       │
                  │  Gateway        │
                  │  (port 18789)   │
                  └────┬────────┬───┘
                       │WS      │HTTP /api/chat/media/*
                       │native  │(proxied through /media)
                  ┌────▼────────▼─────────────────────────┐
                  │  iClaw Express + WS app (port 3000)   │
                  │                                       │
                  │  gatewayWs.ts    ← single WS bridge,  │
                  │      ▲           ← tick watchdog,     │
                  │      │           ← onReconnect()      │
                  │  openclawWs.ts   ← typed RPC client   │
                  │      ▲                                │
                  │      ├─ chatRunner.ts (per-turn)      │
                  │      ├─ gatewayEvents.ts (global)     │
                  │      ├─ scheduler.ts (cron sweep)     │
                  │      └─ projectMemory.ts (facts)      │
                  │      ▲                                │
                  │      ├─ routes/ws.ts (browser WS)     │
                  │      ├─ routes/chats.ts (HTTP forms)  │
                  │      ├─ routes/projects.ts            │
                  │      └─ routes/gateway.ts (proxies)   │
                  │                                       │
                  │  wsHub.ts ← pub/sub fanout            │
                  └────┬───────────────────────────────┬──┘
                       │WS /ws (real-time)             │HTTP (forms + pages)
                       ▼                               ▼
                  ┌────────────────────────────────────────┐
                  │  Browser — public/js/iclaw.js +        │
                  │  vendored marked.min.js + highlight.js │
                  └────────────────────────────────────────┘
```

Browser ↔ server is **one persistent WebSocket** at `/ws`. Chat traffic (send, abort, exec approval, streaming turn events, cross-tab sync) flows through it. HTTP routes exist for page rendering (EJS), form actions, the `/media/*` proxy, and JSON endpoints under `/api/gateway/*`. Every HTTP mutation that touches a chat also emits the matching `chat-updated` / `chat-deleted` over WS so other tabs catch up instantly.

### Runtime modes (Work / Safe work / Incognito)

The default **Full Power** mode goes to the gateway (above). The other three modes
are served by the bundled **iClaw runtime** (`packages/iclaw-runtime`), a small
agent runtime kept deliberately minimal:

- The host calls it over HTTP on `127.0.0.1:7430` (`src/services/workRuntime.ts` →
  runtime `index.ts`). The model loop runs **on the host** via OpenRouter.
- Tool/shell execution is isolated in a **Docker sandbox**, one model per file:
  `secure-runner.ts` (Safe work / Incognito — everything runs in a per-turn
  container) and `work-container.ts` (Work — agent loop + file tools on the host,
  only `run_command` containerized with per-folder `:ro`/`:rw` mounts). Both share
  ONE image, `container/secure-sandbox.Dockerfile` (tag `iclaw-secure:latest`,
  built via `npm run build:secure-image`); `node:22` is only an emergency fallback.
- The live runtime is just 9 files (`index.ts`, `sessions.ts`, `secure-runner.ts`,
  `work-container.ts`, `agent/{loop,tools,security,prompt-dump}.ts`, `log.ts`).
  Path/secret enforcement lives in `agent/security.ts`. Needs Docker + an
  OpenRouter key; without either, these modes are unavailable and fall back to
  Full Power. This is NOT the OpenClaw path — keep the two cleanly separate.

### The canonical client paths

| What you want | Use |
| --- | --- |
| Run a turn (high-level: persist + broadcast) | `chatRunner.sendMessage({chatId?, content, agentLabel?, projectId?, subscriber?})` |
| Push to a subscribed chat | `wsHub.broadcastToChat(chatId, msg)` |
| Push to every connected tab | `wsHub.broadcastAll(msg)` |
| List configured agents | `openclawWs.listAgents()` |
| List configured models for the picker | `openclawWs.listModels('configured')` |
| List slash commands for autocomplete | `openclawWs.listCommands({agentId})` |
| Create a session | `openclawWs.createSession({ agentId })` |
| Delete a session (on chat delete) | `openclawWs.deleteSession(key)` |
| Send + stream a single OpenClaw turn (low-level) | `openclawWs.runTurn({ sessionKey, message, onEvent })` |
| Get transcript | `openclawWs.getHistory(sessionKey)` |
| Cancel running turn | `openclawWs.abortRun(sessionKey, runId?)` |
| Resolve an exec approval | `openclawWs.resolveExecApproval({approvalId, decision, reason?})` |
| Change per-session model | `openclawWs.patchSession({key, model})` |
| Today's gateway spend | `openclawWs.usageCost({from, to})` |
| Subscribe to sessions.changed | `openclawWs.subscribeSessions()` |
| Generic RPC against the gateway | `gatewayWs.request(method, params)` |
| Raw frame access | `gatewayWs.onFrame(listener)` |
| Run a callback on every fresh hello-ok | `gatewayWs.onReconnect(listener)` |
| Read base URL for templates / proxy targets | `openclaw.baseUrl` |

Anything that mutates a chat from a route handler MUST also broadcast the corresponding `chat-updated` / `chat-deleted` over `wsHub.broadcastAll`. `routes/chats.ts` POST handlers do this directly; `chatRunner` does it for title / agent changes inside a turn.

`src/services/openclaw.ts` is a deliberately tiny module — just `baseUrl` + `health()`. **Do not grow it.** All chat / agent / session work goes through `openclawWs.ts`.

### Event flow during a turn

A `runTurn` call emits these `TurnEvent`s through `onEvent`:

- `text-delta` — streaming text chunks (use directly as token stream).
- `tool-start` / `tool-end` — agent tool calls (bash, file, etc.) with a human label and optional `detail` for the actual command.
- `lifecycle` — phase transitions. Terminal phases (`end`/`error`/`aborted`/`cancelled`/`failed`/`terminated`/`stopped`) all unwind the lock.
- `attachment` — `{ url, mime, label? }`; URL is rewritten to `/media/...` so the browser fetches through our proxy without ever seeing the gateway token.
- `reasoning` — analysis-stream items (model chain-of-thought). Only forwarded to the client when `chats.reasoning_mode != 'off'` for that chat.
- `text-final` — emitted once at end, with the canonical final text.

`runTurn` resolves on one of three signals, in priority order:

1. **`chat:state=final`** — normal completion. Canonical terminator.
2. **`onAbort` callback** — fired from `abortRun`'s `chat.abort` RPC response when the gateway acks `aborted: true`. Race-proof against event/response ordering on the same socket.
3. **Post-`lifecycle:end` grace** — if `state:final` never follows `end` within `POST_END_FINAL_GRACE_MS` (15s), we settle with whatever stream text we have rather than wait out the 60-min upper-bound timeout. Logs a warning.

Non-'end' terminal lifecycle phases (`error`/`aborted`/`cancelled`/`failed`/`terminated`/`stopped`) reject the promise.

After resolution (non-aborted), `runTurn` does one `chat.history` fetch and slices from the LAST `user` row forward — the slice is the current turn's events. `resolveFromHistorySlice` (in `turnReply.ts`) picks the canonical reply: prefers the `message` tool's `sourceReply.text`, falls back to the most recent assistant text in the slice. This is returned as `authoritativeText`. `chatRunner` prefers it over the streamed buffer.

Aborted turns intentionally skip the history slice — the gateway hasn't appended a fresh assistant row, so a fetch would return stale text from the previous turn.

**Stop-during-`chat.send` race:** abort RPCs that arrive while `chat.send` is in flight (gateway has no run yet → `aborted: false`) record an intent in `pendingAbortIntents`. Once `chat.send` returns with a runId, `runTurn` consumes the intent and re-issues the abort.

### Session keys

OpenClaw owns session keys. They look like `agent:main:dashboard:<uuid>`. Our DB column `chats.openclaw_session_id` stores this exact string after the first message. `ensureSession(chatId)` creates a session on demand for any chat that doesn't have a real (`agent:…`) key yet — anything not starting with `agent:` is replaced silently. Don't add classification helpers.

### Status & reload recovery

`chatStatus` tracks per-chat lock + current activity in memory. When the browser reloads mid-turn, the EJS template renders a reload placeholder with the snapshot, and the client adopts it via `ensureStreamEl()` so the next event continues to flow into the right node.

Sidebar staleness across tabs is handled by `chats.updated_at` updates being broadcast as `chat-updated`. A SQLite trigger (`trg_chats_touch_on_message`) keeps `chats.updated_at` correct on every message insert, so a forgotten `chats.touch()` can't break sidebar ordering.

### Gateway events module (`gatewayEvents.ts`)

Background broadcasts from the gateway:

- `sessions.changed` → re-broadcast as `gateway-session-changed` (informational).
- `exec.approval.requested` → matched to an iClaw chat via `chats.findBySessionKey` and pushed as `exec-approval-requested`.
- `exec.approval.resolved` → pushed as `exec-approval-resolved` (also fires when another client resolves).
- `health` / `shutdown` → folded into a derived `gateway-status` (`ok` / `degraded` / `shutdown`) that drives the header badge.

This module also re-fires `sessions.subscribe` on every `gatewayWs.onReconnect` so a WS bounce (laptop sleep/wake) doesn't leave us deaf.

### Cloud share (`public/js/share.js` + `ICLAW_CLOUD_URL`)

Companion to [iClaw-cloud](https://github.com/iClawApp/iClaw-cloud). `loadCloudShareBaseUrl()` in `src/services/config.ts` supplies the POST target. **When `ICLAW_CLOUD_URL` is unset**, it defaults to `https://app.iclaw.digital`. Set to `0`, `false`, `off`, `no`, or `disabled` (case-insensitive) to hide the **Share** button. The modal is driven by `public/js/share.js`; crypto runs in the browser:

1. `fetch('/chats/:id/messages')` to get the canonical transcript.
2. JSON.stringify → gzip via `CompressionStream` → AES-256-GCM with a random 256-bit key + 96-bit nonce.
3. If password protection is on, derive a wrap-key via `PBKDF2-SHA256(200_000)` from the password + a random 16-byte salt, then AES-GCM-wrap the real key.
4. POST `{ ciphertext, nonce, salt?, wrappedKey?, hasPassword, ttlDays, maxViews? }` (all binary base64) to `${ICLAW_CLOUD_URL}/api/shares`. The iClaw server is NOT on the path — the browser talks to iClaw-cloud directly. CORS must allow iClaw's origin on the cloud side.
5. The returned share URL is `${cloudUrl}/s/<id>` plus a `#k=<base64url(key)>` fragment when there's no password. With a password, the URL has no fragment.

iClaw never sees the symmetric key, the password, or the plaintext after step 1. The cloud server stores ciphertext only.

### Project memory (`projectMemory.ts`)

- `buildGatewayUserMessage(content, projectId)` prepends a `[Project context]` block of facts under a ~1500-token budget. The user message stored in iClaw is always the raw text; only the gateway sees the augmented version.
- After every assistant reply, `scheduleProjectFactExtraction` runs an LLM sub-request (against `openclaw/default`, regardless of the chat's agent) to propose 0–3 short facts. Suggestions land in `project_fact_suggestions`; the user accepts/rejects each inline.
- At 30+ facts, `compactProjectFacts` merges down to 15 via another LLM call.

OpenClaw has its own per-agent memory (`MEMORY.md`, `memory/<date>.md` in `~/.openclaw/`). The iClaw project facts are a **separate layer at a different scope** — per project, across multiple chats and potentially different agents. They do not duplicate OpenClaw memory.

## Stack rules

- **Node.js 20+**, TypeScript strict mode. No `any` without a comment explaining why.
- **Express + EJS** server, vanilla JS + plain CSS client. `marked` and `highlight.js` are the only client-side runtime deps.
- **better-sqlite3** for storage. Synchronous on purpose — keeps request handlers simple.
- **No frontend framework, no build step beyond `tsc`.** If you genuinely need reactivity, talk first.
- **CSS uses the design system at the top of `public/css/style.css`.** Tokens (`--space-*`, `--radius-*`, `--shadow-*`, `--z-*`, semantic colours like `--warn`/`--approve`/`--danger`) + primitives (`.btn`, `.chip`, `.card`, `.menu`). Reach for primitives before adding new snowflake selectors. Hardcoded hex is a code smell.
- **Tests live under `test/`.** New features should land with at least one test (unit if pure, integration if it touches routes / DB / chatRunner). The vitest setup pins `DB_PATH` per worker so tests never share state.

## Conventions

- Routes return JSON when `Accept: application/json` or for `/api/*` paths; otherwise EJS or 302 redirect for forms.
- Every commit must pass `npm run typecheck && npm test && npm run build` (CI runs all three).
- One logical change per commit. Imperative subject ≤ 72 chars.
- See `CONTRIBUTING.md` for the rest of the rules.

## Things that have already burned us

- **Adding "lazy legacy migration" code with detection branches.** Removed long ago — anything not a real OpenClaw key is just replaced. Don't re-add classification helpers.
- **Asking OpenClaw for AI-generated titles in the same session as the chat.** Pollutes the transcript. We use a throw-away session per title attempt (`chatTitle.ts`).
- **Sub-tasks running against the chat's agent.** Fact extraction + compaction now always go through `openclaw/default` — keeps the two pipelines consistent and avoids specialised agents underperforming on a plain text-extraction prompt.
- **Treating `kind: 'analysis'` items as tool calls.** That's reasoning. We emit it as a separate `reasoning` event and the receiver decides whether to forward.
- **Resolving `runTurn` on `lifecycle:end` alone, without grace.** Races the canonical `chat:final` payload. We resolve on `state:final` first; if `end` fires and `final` doesn't follow within `POST_END_FINAL_GRACE_MS`, we settle with the stream buffer + log — never silently hang.
- **Reading `chat.history` without a turn-scoped slice.** The old `canonicalAssistantText` walked global history backwards to the first `user` row, which leaked across turn boundaries when the current user row wasn't yet committed. Use `sliceFromLastUser` + `resolveFromHistorySlice` from `turnReply.ts`.
- **Bypassing `gatewayWs.subscribeSession`.** Calling `request('sessions.messages.subscribe', …)` directly skips the `subscribedSessions` bookkeeping, and a WS reconnect mid-turn won't re-subscribe — the turn goes deaf and hangs.
- **Recording an abort intent when no `runTurn` is active.** Would silently kill the next turn the user starts. `abortRun` only sets the intent when a callback is registered.
- **Sending content as a string array via OpenAI-compat.** OpenClaw's native protocol takes `chat.send { sessionKey, message: string, idempotencyKey }`. Don't try to mimic OpenAI message arrays.
- **Hardcoding the bearer token anywhere.** It must be read at runtime via `loadOpenClawConfig()`. Never log it.
- **Forgetting to re-subscribe on WS reconnect.** Use `gatewayWs.onReconnect(...)` for any global subscription (the per-session ones are re-applied automatically).
- **Hard-coding hex colours in component CSS.** Use the design tokens. Adding a new token is fine; bypassing them is not.

## When you're stuck

- Read the protocol doc inside the installed `openclaw` npm package: `~/.nvm/versions/node/<v>/lib/node_modules/openclaw/docs/gateway/protocol.md`. It has the full WS RPC surface (sessions, agents, tools, talk, exec approvals, cron, presence, …).
- The OpenClaw control-ui bundle (`http://127.0.0.1:18789/assets/index-*.js`) shows how their own dashboard uses each RPC.
- Vitest mocks for the OpenClaw client live in `test/helpers/gatewayStubs.ts` — they are the cheapest way to script gateway behaviour without a real backend.

That's it. Keep iClaw small, focused, and OpenClaw-native.

---
> Source: [iClawApp/iClaw](https://github.com/iClawApp/iClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
