## widgetdc-orchestrator

> You are working on WidgeTDC Orchestrator — multi-agent coordination layer.

You are working on WidgeTDC Orchestrator — multi-agent coordination layer.
TypeScript orchestration service: agent registry, chain engine, cognitive proxy, LLM proxy, MCP bridge.
Tech: Node 20+, Express, TypeScript strict, ESM only, esbuild → dist/
Production: https://orchestrator-production-c27e.up.railway.app

Key files:
- `src/index.ts` — entry point
- `src/agent-registry.ts` — agent CRUD + Redis persistence
- `src/chain-engine.ts` — sequential/parallel/loop/debate chains
- `src/cognitive-proxy.ts` — RLM Engine HTTP proxy
- `src/llm-proxy.ts` — multi-provider LLM proxy
- `src/routes/` — API routes
- `frontend/index.html` — Command Center SPA (vanilla JS)
- `dist/` — pre-built bundle (committed)

Rules:
- ESM only — import/export, never require()
- TypeScript strict mode
- Auth: Authorization Bearer on all backend/RLM calls
- dist/ is committed — Railway runs `node dist/index.js` directly
- Frontend is vanilla JS — NO TypeScript syntax
- MCP route format: {"tool":"name","payload":{...}} — ALDRIG args
- Read before write: læs 2 eksisterende filer i mappen først
- Conventional commits (feat:, fix:, docs:, refactor:)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Clauskraft) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
