## web-world-models

> **Stack:** Node 20+, TypeScript, Express, Vite + React (TS), OpenRouter for LLM text & image generation, JSON storage (no DB).

**Stack:** Node 20+, TypeScript, Express, Vite + React (TS), OpenRouter for LLM text & image generation, JSON storage (no DB).

## Integration summary
- Theme plugins implement `AgentPlugin` (`server/src/plugins/agentPlugin.ts`) and are registered in `themeService.ts` via `PluginRegistry`.
- Providers: `OpenRouterPlugin` (preferred when `OPENROUTER_API_KEY` exists) + `StaticThemePlugin` (palette backfill when OpenRouter is offline). Provider readiness is exposed at `/api/themes/providers` for the HUD badges.
- World beacons are procedurally generated; seeds come from `worldPromptService.ts` and on-demand grid nodes from `proceduralBeaconService.ts`. Dataset tags vary (`procedural_seed`, `procedural_geo`).
- Guide agent (`/api/agent/generate`) and travel-plan agent (`/api/agent/generateTravelPlan`) both expect OpenRouter. Plans auto-fallback to local templates; guides require a valid key.
- Scene prompts (`/api/ai/scene`) use OpenRouter when available, otherwise return static page presets.

## Quick dev cues
- Set `OPENROUTER_API_KEY` and optional `OPENROUTER_MODEL` in `server/.env`.
- Core entrypoints: `server/src/services/themeService.ts`, `contentAgentService.ts`, `travelPlanAgentService.ts`, `sceneImageService.ts`.
- Add a provider: implement `AgentPlugin`, import/register in `themeService.ts`; readiness will appear automatically in `/api/themes/providers`.

---
> Source: [Princeton-AI2-Lab/Web-World-Models](https://github.com/Princeton-AI2-Lab/Web-World-Models) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
