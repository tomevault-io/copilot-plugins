## ima2-gen

> Local image generation studio (v2.0.1) — CLI + 웹 UI

# ima2-gen — AI Context

## What This Project Does
Local image generation studio (v2.0.1) — CLI + 웹 UI
- GPT OAuth, API Key, Grok, Gemini API, Antigravity CLI 다중 provider 지원
- 텍스트→이미지, 이미지→이미지(편집), 비디오 생성
- SSE 멀티플렉싱: 단일 `GET /api/events` SSE 채널 + async POST (202) 아키텍처
- 병렬 생성 (최대 12건, 브라우저 연결 포화 없음)

## Tech Stack
- Runtime: Node.js >=20 (ES Module)
- Server: Express 5
- API Client: OpenAI SDK v5
- OAuth: openai-oauth (ChatGPT 세션 프록시)
- Grok: bundled progrok (xAI Images API)
- Gemini: Google Generative Language API / Vertex AI
- Frontend: React + Vite (`ui/src`, built to `ui/dist`)
- SSE: lib/eventBus.ts (ring buffer pub/sub) + routes/events.ts

## Project Structure
```
ima2-gen/
├── bin/                  # CLI entry + subcommands
├── server.ts             # Express bootstrap / static UI serving
├── config.ts             # Runtime config
├── routes/               # API route modules (*.ts source)
│   ├── events.ts         # GET /api/events SSE multiplexing
│   ├── multimode.ts      # Multimode batch (async POST + dual-emit)
│   ├── nodes.ts          # Node mode (async POST + dual-emit)
│   ├── video.ts          # Video generation (async POST + dual-emit)
│   └── ...               # generate, edit, sessions, history, etc.
├── lib/                  # Server helpers (*.ts source)
│   ├── eventBus.ts       # Global pub/sub ring buffer (2000 events)
│   ├── ssePublish.ts     # Cancel-done race guard
│   ├── inflight.ts       # Job lifecycle tracking
│   └── ...               # OAuth, storage, sessions, etc.
├── ui/src/               # React/Vite app source
│   ├── lib/eventChannel.ts  # Singleton EventSource for /api/events
│   ├── lib/sseStreamError.ts # SSE error parser
│   └── store/store*Impl.ts  # Modular Zustand slices
├── ui/dist/              # Built frontend served by server.js
├── site/                 # Astro marketing/docs site (GitHub Pages)
├── integrations/comfyui/ # ComfyUI bridge/custom node
├── structure/            # Current architecture reference docs (00-07)
├── devlog/               # _plan (active), _fin (archived)
├── tests/                # node:test contracts/regressions (968 cases)
└── package.json
```

## Devlog Phase Roadmap
- Current active plans live under `devlog/_plan/`.
- Completed plans live under `devlog/_fin/`.
- Legacy phase docs live under `devlog/_plan/_legacy/`.
- Use `structure/07-devlog-map.md` and `devlog/_plan/README.md` as the current roadmap references.

## Conventions
- ES Module only (import/export)
- File length < 500 lines (split if exceeded)
- Function length < 50 lines
- try/catch mandatory for all async operations
- Config values in config.js or .env, never hardcode

## Test Command
```bash
npm run typecheck          # tsc --noEmit (server + lib)
npm run typecheck:tests    # tsc --noEmit (test files)
npm test                   # node:test (968 cases)
npm run test:inventory     # verify test file registry
cd ui && npm run build     # Vite production build
```

## Heartbeat
- 20분마다 devlog/_plan 점검 및 다음 작업 제안
- 완료된 phase는 _fin/으로 이동 (YYMMDD_ prefix)

---
> Source: [lidge-jun/ima2-gen](https://github.com/lidge-jun/ima2-gen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
