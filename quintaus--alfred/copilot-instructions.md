## alfred

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# JARVIS dashboard — project notes

A JARVIS-style HUD that uses Claude (Anthropic API) as the brain. Built in
phases (see README "Roadmap"). **Phases 1–5 are done (HUD, Claude brain, Google,
voice, Spotify/weather/search).**

Key conventions:

- **Stack:** Next.js 16 App Router + TS, Tailwind v4 (CSS `@theme` in
  `src/app/globals.css`), Motion (`motion/react`), Zustand, lucide-react,
  `@anthropic-ai/sdk`.
- **The brain (Phase 2):** `src/app/api/command/route.ts` runs a manual
  streaming agentic loop and emits Server-Sent Events (`ServerEvent` union in
  `types.ts`). `src/lib/useCommand.ts` consumes the stream into the store.
  Persona = `src/config/persona.ts`. Tools = `src/lib/tools/index.ts` (one
  `ToolSpec` per capability). Claude client + model id live in
  `src/lib/server/anthropic.ts` (server-only). The route runs with no key too
  (`runDemo` fallback) and emits `data` events to push fresh widget data.
- **Google (Phase 3):** OAuth routes in `src/app/api/auth/*`; client/session in
  `src/lib/server/google.ts`; API→shape adapters in `google-data.ts`; encrypted
  token store in `token-store.ts`; scopes in `src/config/google.ts` (read-only +
  calendar event create). Tool handlers use real Google when connected
  (`getAuthedClient`), else mock. Tokens encrypted under `.data/` (git-ignored).
  NOTE: `googleapis` pulls two `google-auth-library` versions → an OAuth2Client
  type clash; resolved via `overrides.google-auth-library` in package.json (keep
  it). Type the auth client as `Auth.OAuth2Client` from `googleapis`.
- **Voice (Phase 4):** `src/components/hud/VoiceProvider.tsx` owns STT + TTS
  (Web Speech API), live mic amplitude → store `inputLevel`, and the Porcupine
  wake word. Wake word is opt-in (`NEXT_PUBLIC_PICOVOICE_ACCESS_KEY` +
  `porcupine_params.pv` in `public/`); push-to-talk + text always work. Picovoice
  modules are dynamically imported (browser/WASM only) and typed `any` to keep the
  build green. All browser APIs live in effects/handlers (SSR-safe).
- **Integrations (Phase 5):** `lib/server/weather.ts` (Open-Meteo, keyless),
  `web-search.ts` (Brave if `BRAVE_SEARCH_API_KEY`, else keyless DuckDuckGo),
  `spotify.ts` (OAuth + playback, mirrors google). `token-store` is now
  per-provider (`saveSession(provider, …)`); `/api/auth/status` returns
  `{google, spotify}`. Demo mode runs the REAL tool handlers and emits `data`
  events, so live data shows without an Anthropic key. ElevenLabs hook: `/api/tts`
  + `NEXT_PUBLIC_USE_ELEVENLABS`. External APIs (open-meteo, duckduckgo, spotify,
  elevenlabs) are 403-blocked in the web sandbox — verify via build + graceful
  fallback, not live calls.
- **Claude usage:** model `claude-opus-4-8`, adaptive thinking, NO
  `temperature`/`budget_tokens`/prefill (they 400). Before touching Anthropic
  code, consult the `claude-api` skill — don't code SDK usage from memory.
- **State** lives in `src/lib/store.ts` (`useJarvis`). `status` + `inputLevel`
  drive the reactor/visualizer; `focusedWidget` + `highlightedIds` are the
  targets of Claude's UI directives; `transcript`/`streamingText`/`activities`
  hold the conversation.
- **Data shapes** are in `src/lib/types.ts`; mock data in `src/lib/mock-data.ts`.
  Real tool handlers (Phase 3+) must return these same shapes so widgets are
  source-agnostic.
- **Theme tokens** (colours/fonts) are all at the top of `globals.css`.
- **Security:** secrets are server-only (`src/lib/server/`); never import API
  keys into client components. `.env*` (except `.env.example`) is git-ignored.
- Anything using the browser (canvas, `window`, mic) must be a `"use client"`
  component and touch browser APIs only inside effects (SSR-safe).
- Verify with `npm run build` (type-checks + builds). Test the API loop without a
  key via demo mode: `curl -N -X POST localhost:3000/api/command -d '{"message":"..."}'`.
  Browser screenshots: Playwright's CDN is blocked, but Puppeteer works (its
  Chrome downloads from Google's CDN). Install puppeteer in /tmp, launch with
  `--no-sandbox`, and screenshot localhost.

---
> Source: [QuintAus/Alfred](https://github.com/QuintAus/Alfred) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
