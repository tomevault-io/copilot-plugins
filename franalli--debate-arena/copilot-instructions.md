## debate-arena

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Debate Arena — a React web app that orchestrates real-time debates between three AI agents (Advocate/Google Gemini, Critic/OpenAI, Wildcard/Anthropic Claude) across 3 rounds on a user-submitted topic. The Wildcard judges each round. Results are visualized as a D3 force-directed graph.

## First-time setup

1. `brew install gitleaks vercel` — `gitleaks` backs the pre-commit hook; `vercel` is the local dev server.
2. `npm install` — installs deps and activates the gitleaks hook via the `prepare` script.
3. `cp .env.example .env.local` and fill in every required value. `.env.local` is gitignored; never commit real values.
4. `vercel link` — one-time, connects this clone to the Vercel project.
5. `vercel dev` — runs frontend + serverless API on `http://localhost:3000`.

## Commands

```bash
vercel dev         # Local dev: serves frontend + /api/* on one port (http://localhost:3000)
npm run dev        # Vite-only dev server (http://localhost:5173) — /api/* will 404
npm run build      # Production build → dist/
npm run lint       # ESLint
```

Local development uses the Vercel CLI (`vercel dev`) so the Vite frontend and the serverless functions in `api/` run together. `npm run dev` alone won't serve `/api/*` — there's no Vite proxy and no separate Express backend.

No test suite is configured.

## Architecture

**Frontend**: React 19 + Vite 8, no TypeScript, no state management library. All state lives in App.jsx via hooks.

**Backend**: Vercel serverless functions in `api/`. Two parallel per-claim pipelines, plus a verdict and a debate-cache:

- **Streaming** (`POST /api/debate-stream`, used by MSE-capable clients): streams LLM SSE tokens → `api/_chunker.js` sentence chunker → serial ElevenLabs `streamWithTimestamps` with `previousText` for prosody continuity → NDJSON (`chunk_meta` / `audio` / `claim_complete` / `error`) to client. Agents emit a `TEXT:\n<prose>\n---META---\n{...}` format so the chunker can consume raw prose tokens without parsing JSON. Cache namespaces: `getCachedLlm` (shared with legacy path) + `ttsStreamCacheKey` (isolated NDJSON blob).
- **Legacy** (`POST /api/debate` JSON + `POST /api/tts` NDJSON, used by iOS Safari with no MediaSource): two-step LLM→TTS, single-shot per claim. Both endpoints are **permanent** — do not delete; they're also the verdict TTS path.
- `POST /api/verdict` (JSON): wildcard's final judgement. Always non-streaming (~150 tokens, not worth the work).
- `GET/POST /api/debate-cache`: full-debate cache check + persistence.

Shared modules in `api/`: `_shared.js` (provider LLM calls + streaming variants, rate limit, validation, cache helpers), `_prompts.js` (templates + `BEHAVIOR_HASH` invalidation), `_chunker.js` (`SentenceChunker`), `_streaming.js` (TEXT/META state machine + parsers), `_tts.js` (EL client + voice config).

**Data flow**: TopicInput → `runDebate()` in `src/lib/debate.js` → dispatches via `hasMSE()`:
- `liveGenStreaming()`: per claim, opens `/api/debate-stream` and consumes NDJSON via `startClaimStream` in `src/lib/audio.js`. Each claim's network request pipelines behind the previous claim's *audio* playback (gated via `gateBeforePlay`); audio remains strictly serial.
- `liveGenLegacy()`: per claim, `callAgent` (`/api/debate`) then `speakClaim` (`/api/tts`) — identical to pre-refactor.

Claims feed `buildGraphData()` (src/lib/graphUtils.js) → DebateGraph renders via D3.

**Key modules**:
- `src/lib/agents.js` — parser for both TEXT/META prose-trailer and legacy JSON formats. Claim IDs follow `{prefix}_r{round}_{index}` (e.g., `adv_r1_1`).
- `src/lib/debate.js` — orchestrator with `hasMSE()` capability branch. Cache replay path + verdict path shared across both branches.
- `src/lib/audio.js` — `playAudioStream` (legacy single-shot, used by verdict + iOS) and `startClaimStream` (multi-chunk envelope, gated playback for pipelining, cumulative karaoke alignment offset).
- `src/lib/graphUtils.js` — Graph data builder, scoring (`computeWildcardScore`), round winner logic.
- `src/components/DebateGraph.jsx` — D3 SVG graph (800×700 viewBox). Fixed agent anchors: Advocate top-center, Critic bottom-left, Wildcard bottom-right.

**Agent colors**: Green (Advocate), Red (Critic), Purple (Wildcard).

**Wire protocol**:

LLM prompt output format (all three claim agents — see `api/_prompts.js`):
```
TEXT:
<one paragraph of prose>
---META---
{"rebuts": "crt_r1_1" | null, "agrees_with"?: "wld_r1_2"}
```

`/api/debate-stream` NDJSON events (one claim per request):
- `{type:"chunk_meta", seq, chunkText}` — emitted before each TTS chunk
- `{type:"audio", seq, audioBase64, alignment}` — EL frames (alignment timestamps are relative to the chunk; client accumulates a cumulative offset for karaoke)
- `{type:"claim_complete", fullText, rebuts, agrees_with}` — emitted once, before `res.end()`
- `{type:"error", recoverable:false, message}` — on mid-stream failure

Client mirror lives in `src/lib/audio.js` `startClaimStream`. Both sides of the protocol use literal strings (Vite can't import from `api/`); rename one, rename both.

## Environment Variables

Set in `.env.local`: `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GOOGLE_API_KEY`. Optional model/token overrides per provider (e.g., `ANTHROPIC_MODEL`, `OPENAI_MAX_TOKENS`). Rate limit config: `RATE_LIMIT_IP_DAILY`, `RATE_LIMIT_GLOBAL_DAILY`, `DEBATE_COOLDOWN_MS`.

## Secrets discipline

- **Never reproduce live values** from `.env.local`, environment variables, or any other secret source in any generated file (planning docs, READMEs, code comments, examples, error messages, AI-generated artifacts). Always use placeholders:
  ```
  ELEVENLABS_API_KEY="<your-key>"
  KV_REST_API_TOKEN="<your-token>"
  ```
  If a planning artifact needs to show what `.env.local` looks like, show the **shape**, not the contents.
- **Plan / spec / design artifacts must be deleted after they're consumed.** A planning skill that drops a doc in `docs/` (or anywhere) should clean it up once the implementation is committed. `docs/` is gitignored so artifacts can't reach the repo, but they're still local-only context that rots fast and tends to collect copy-pasted env content. Delete plan files when the corresponding code lands.
- The `gitleaks` pre-commit hook (`.githooks/pre-commit`, `.gitleaks.toml`) is the mechanical backstop. It blocks commits matching the gitleaks default ruleset plus a custom `sk_[a-f0-9]{40,64}` rule for ElevenLabs keys. Activated via `git config core.hooksPath .githooks`, which `npm install` sets automatically through the `prepare` script. Bypass (`git commit --no-verify`) requires a written reason in the commit message and is reserved for emergencies.
- On a fresh clone: `brew install gitleaks && npm install` — that's the full setup. The hook is inert without the binary and prints a helpful install message if it's missing.

## Gotchas

- **Always `vercel dev`, not `npm run dev`**, when testing `/api/*`. Vite alone doesn't proxy to the serverless functions.
- **The chunker emits zero chunks during streaming in fast mode.** `FAST_MAX_TOKENS=100` produces one short sentence; the chunker only flushes on stream-end. Streaming-tier benefits (TTFA reduction) are deep-mode only.
- **Don't bypass `gateBeforePlay`.** `currentAudio` / `currentResolve` in `src/lib/audio.js` are module-level singletons; the gate-deferred assignment is what prevents two coexisting MSE pipelines from clobbering each other during pipelined streaming.
- **`api/debate.js` and `api/tts.js` are permanent.** iOS Safari has no MediaSource — the orchestrator routes those clients to the legacy two-step path. Also serves the verdict TTS for all clients. Header comments in both files say so; do not delete.
- **The verdict path is always non-streaming.** Different prompt shape (`{winning_arguments, loser_gap}`), only ~150 tokens, chunking would lose more in complexity than it gains in latency.
- **`BEHAVIOR_HASH` (in `api/_prompts.js`) invalidates all caches** when any of: agent template, style snippet, max-rounds, sampling settings, or `buildUserMessage` changes. Edits to those automatically force regeneration on the next visitor — no manual cache wipe needed.

## Notes

- Response parsing in `agents.js` accepts both formats: the current `TEXT:\n…\n---META---\n{...}` prose-trailer (preferred) and legacy `{"claims":[{...}]}` JSON (kept as a fallback for cached entries written pre-refactor and as a safety net for malformed responses).
- Deployed at `https://debate-arena-ten.vercel.app`. Push to `main` → auto-deploys; branch pushes get preview URLs; `vercel --prod` for manual prod deploy. No CORS config needed — API functions are same-origin (Vercel serverless).
- Rate limiting + cooldown + caches are backed by Upstash KV (`@upstash/redis`) via `KV_REST_API_URL` / `KV_REST_API_TOKEN`. Limits are global across serverless instances. When KV env vars are missing, rate limiting fails open (provider-side spend caps are the last line of defense).
- **Time-to-first-audio benchmarks** (round-1-advocate, the only user-visible LLM latency — claims 2-9 are pipelined behind audio):
  - Deep cold (cache miss): ~9s. Dominated by Gemini 3.1 Pro reasoning TTFT (~3-5s); not addressable in app code without switching models.
  - Deep cache HIT (full-debate replay): ~2.4s.
  - Fast cold: ~5-7s.
- **Browser support**: MSE-capable clients (desktop Chrome/Firefox/Safari, Android) use `/api/debate-stream`. iOS (Safari, Brave, and any other iOS browser — Apple forces WKWebView) is routed to legacy `/api/debate` + `/api/tts`. iOS 17.1+ does expose `MediaSource`, but its sourceBuffer quota is small enough that long debates trip `QuotaExceededError` mid-stream and brick playback; `hasMSE()` in `src/lib/audio.js` excludes iOS UAs unconditionally.
- **Code conventions**: no TypeScript, JSX functional components with hooks, `.js` extension explicit in imports, single quotes, 2-space indent, no semicolons.

---
> Source: [franalli/debate-arena](https://github.com/franalli/debate-arena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
