## deep-research-mcp-server

> description: Deep Research MCP Server – Rules and integration guide


---
description: Deep Research MCP Server – Rules and integration guide
---

# Deep Research MCP Server – Rules for Contributors

This document explains repo layout, environment, Gemini usage, search provider priority/fallbacks, and safe contribution steps for users.

- __Repo__: `deep-research-mcp-server/`
- __Stack__: Node.js 22.x, TypeScript 5.x
- __Key libs__: `@google/genai` (Gemini), `@mendable/firecrawl-js`, `@modelcontextprotocol/sdk`, `zod`, `lru-cache`
- __Primary entry points__:
  - __CLI__: `src/run.ts`
  - __MCP server__: `src/mcp-server.ts`
  - __Core logic__: `src/deep-research.ts`
  - __AI wrapper__: `src/ai/providers.ts`
  - __Text splitting__: `src/ai/text-splitter.ts`
  - __Feedback__: `src/feedback.ts`
  - __Output/Progress__: `src/output-manager.ts`, `src/progress-manager.ts`, `src/terminal-utils.ts`
  - __Prompts__: `src/prompt.ts`
  - __Types/Utils__: `src/types.ts`, `src/utils/json.ts`, `src/utils/sanitize.ts`

## Environment & Config

- __Required__: `GEMINI_API_KEY`
- __Optional__: `FIRECRAWL_API_KEY`, `FIRECRAWL_BASE_URL`
- __Config__:
  - `GEMINI_MODEL` (default: `gemini-2.5-flash`)
  - `CONCURRENCY_LIMIT` (default: `5`)

## Gemini SDK Usage (TypeScript)

Use the official Google Gen AI SDK `@google/genai`.

```ts
import { GoogleGenAI } from "@google/genai";

const client = new GoogleGenAI({
  apiKey: process.env.GEMINI_API_KEY!,
  ...(process.env.GEMINI_API_ENDPOINT ? { apiEndpoint: process.env.GEMINI_API_ENDPOINT } : {}),
});
const modelId = process.env.GEMINI_MODEL || "gemini-2.5-flash";
const model = client.getGenerativeModel({ model: modelId, systemInstruction: "You are a precise research assistant." });
```

- __Long context__: prefer `gemini-2.5-flash` for speed; chunk sources via `src/ai/text-splitter.ts` and trim with `adaptivePrompt()` in `src/ai/providers.ts`.
- __Thinking__: enable where useful (models that support it) and cap budgets; surface summaries only.
- __Structured output__: request JSON via schema; validate with `zod`.
- __Function calling / tools__: declare tool schemas and pass in; handle tool responses deterministically.
- __Grounding__: use Google Search Grounding or URL context when needed; keep citations.

### Structured JSON output (concise pattern)

```ts
const response = await model.generateContent({
  contents: [{ role: "user", parts: [{ text: prompt }] }],
  generationConfig: {
    responseMimeType: "application/json",
    responseSchema: {
      type: "object",
      properties: {
        summary: { type: "string" },
        sources: { type: "array", items: { type: "string" } }
      },
      required: ["summary", "sources"]
    }
  }
});
const json = JSON.parse(response.response.text());
```

### Function calling (tools) sketch

```ts
const toolDefs = [{
  name: "searchWeb",
  description: "Query web search",
  parameters: {
    type: "object",
    properties: { query: { type: "string" } },
    required: ["query"]
  }
}];
const toolModel = client.getGenerativeModel({ model: modelId, tools: toolDefs });
```

## Search Provider Priority

- __Primary__: Tavily or Exa (fast, reliable SERP/content). Prefer these first.
- __Fallback__: Firecrawl (`@mendable/firecrawl-js`) when primary is limited. See usage in `src/deep-research.ts`.
- __Notes__: dedupe, normalize, and cache results (`lru-cache`).

## Where to modify things

- __Prompt trimming/dispatch__: `src/ai/providers.ts` (`adaptivePrompt`) and `src/prompt.ts`.
- __Chunking__: `src/ai/text-splitter.ts`.
- __Search orchestration__: `src/deep-research.ts`.
- __Validation__: `zod` models in `src/types.ts` and helpers in `src/utils/json.ts`.

## Contribution Checklist (Safe Changes)

- __Run TypeScript__: `tsc --noEmit` before PRs.
- __Lint/Format__: ensure consistent formatting.
- __Add tests__ for prompt transforms and JSON parsing.
- __Guardrails__: validate model outputs with `zod` and log parse errors.
- __Secrets__: never commit keys; read from environment.
- __Docs__: update this file when changing providers, models, or search order.

## Notes on Models

- Default: `gemini-2.5-flash`.
- Supports structured output, tools, thinking, and long-context use with chunking.
- For very large inputs, prefer batch/chunk + summarize + consolidate.

## Quick Links

- `src/ai/providers.ts` – Gemini calls and token/prompt management.
- `src/deep-research.ts` – pipeline orchestration (search, crawl, synthesize).
- `src/ai/text-splitter.ts` – chunking strategies.
- `src/prompt.ts` – system + templates.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ssdeanx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
