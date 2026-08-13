## vitali-health-copilot

> Vitali Health AI is a mobile-first React + TypeScript + Vite prototype: shared

# Vitali Health AI — Cursor / AI-agent project rules

Vitali Health AI is a mobile-first React + TypeScript + Vite prototype: shared
patient context and real-time coordination for hospital nursing teams.

## Architecture (respect these seams)
- `src/types/models.ts` — the single domain model. Everything speaks these types.
- `src/services/` — the ONLY place that touches "backends":
  - `api.ts` in-memory data store (swap method bodies for real HTTP, keep signatures)
  - `llm.ts` (browser) → calls `/api/llm`; `api/_llm.ts` (server) is the pluggable
    LLM core: Anthropic / Ollama / AnythingLLM, chosen by `LLM_PROVIDER`.
  - `realtime.ts` BroadcastChannel pub/sub · `voice.ts` mic + speech-to-text ·
    `liveData.ts` live vitals stream · `ehr.ts` chart pull · `analytics.ts` metrics
- `src/store/AppContext.tsx` — reactive layer over services. Screens use `useApp()`;
  they never import services directly (except read-only ones like `llm`/`ehr`).
- `src/components/` reusable UI · `src/screens/` routed screens · `src/styles/`
  design tokens + CSS (no UI framework).

## Conventions
- Keep the design tokens in `src/styles/tokens.css` as the source of truth; reuse
  existing utility classes before inventing new ones.
- Every backend call must degrade gracefully to a local mock so the demo never breaks.
- Never put a provider key in the browser bundle — all model calls go through `/api/llm`.
- No real PHI. All patient data here is invented.

## Useful commands
- `npm run dev` · `npm run build` · `npm start` (self-hosted server)
- `docker compose up --build` for the fully local AI stack (app + Ollama + AnythingLLM)

---
> Source: [muza-bug/-vitali-health-copilot](https://github.com/muza-bug/-vitali-health-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
