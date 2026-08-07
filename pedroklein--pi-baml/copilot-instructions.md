## pi-baml

> Pi extension that bridges [BAML](https://github.com/BoundaryML/baml) (a structured output DSL for LLMs) with Pi's provider system. Enables typed LLM function calls via 3 model tiers (light/standard/heavy).

# AGENTS.md — pi-baml Project Guide

## Overview

Pi extension that bridges [BAML](https://github.com/BoundaryML/baml) (a structured output DSL for LLMs) with Pi's provider system. Enables typed LLM function calls via 3 model tiers (light/standard/heavy).

## Quick Context

- **What:** npm package (`pi-baml`) installable via Pi's package system
- **Why:** Typed, reliable structured output from LLMs without manual JSON parsing
- **How:** 3 model tiers configured in settings.json, resolved via Pi's ModelRegistry at call time

## Configuration

```json
{
  "baml": {
    "models": {
      "light": "github-copilot/claude-haiku-4.5",
      "standard": "github-copilot/claude-sonnet-4.6",
      "heavy": "hai-proxy/anthropic--claude-4.6-opus"
    }
  }
}
```

## Repo Structure

```
pi-baml/
├── src/
│   ├── index.ts              ← Extension factory (entry point)
│   ├── eventbus.ts           ← createPiBamlLibrary (stateless, no session_start)
│   ├── lib/
│   │   ├── types.ts          ← All shared types (zero logic)
│   │   ├── config.ts         ← parseBamlSettings()
│   │   ├── bridge.ts         ← resolveModelTier() — single resolution function
│   │   ├── executor.ts       ← createBamlExecutor() → BamlExecutor
│   │   ├── registry.ts       ← FunctionsRegistry, parseFunctionDeclarations
│   │   ├── cache.ts          ← RuntimeCache<T> (SHA-256 content hash)
│   │   ├── readme-parser.ts  ← parseReadmeDescription(), parseReadmeBody()
│   │   ├── type-parser.ts    ← parseTypeDefinitions()
│   │   └── system-prompt.ts  ← renderBamlSystemPrompt()
│   └── tools/
│       ├── baml-list.ts      ← createBamlListTool(registry)
│       ├── baml-run.ts       ← createBamlRunTool(registry, factory, settings)
│       └── baml-exec.ts      ← createBamlExecTool(settings, factory)
├── skills/
│   └── baml/
│       └── SKILL.md          ← BAML authoring skill for the agent
├── examples/                 ← Teaching examples (.baml files)
├── tests/
│   ├── unit/                 ← Unit tests (no network)
│   └── integration/          ← Real BAML runtime tests
├── docs/
│   ├── configuration.md      ← Settings.json reference
│   └── adr/                  ← Architecture Decision Records
├── package.json
├── tsconfig.json
├── tsup.config.ts
├── vitest.config.ts
└── README.md
```

## Key Technical Details

### Model Tiers

The entire model resolution is one function:

```typescript
resolveModelTier(settings, modelRegistry, tier = "standard") → { clientRegistry, bamlProvider }
```

1. Read `settings.models[tier]` → `"github-copilot/claude-haiku-4.5"`
2. `modelRegistry.find("github-copilot", "claude-haiku-4.5")` → Model object
3. `modelRegistry.getApiKeyAndHeaders(model)` → auth
4. Build `ClientRegistry` with "PiClient" as primary

### GitHub Copilot Auth (ADR-013)

BAML 0.85.0's `anthropic` provider sends auth via `x-api-key`, but GitHub Copilot requires `Authorization: Bearer`. The bridge handles this:

- **Anthropic models**: Injects `Authorization: Bearer <token>` in headers, sets `api_key` to `"not-used"`
- **OpenAI models**: Passes real token as `api_key` (BAML's `openai-generic` natively uses Bearer)
- **Always injects**: `X-Initiator`, `Openai-Intent`, `anthropic-dangerous-direct-browser-access`, `accept`

### BAML Runtime API

```typescript
import { BamlRuntime, ClientRegistry, Collector } from "@boundaryml/baml";

const runtime = BamlRuntime.fromFiles("/", { "main.baml": source }, {});
const ctx = runtime.createContextManager();

const cr = new ClientRegistry();
cr.addLlmClient("PiClient", "anthropic", {
  model: "claude-haiku-4.5",
  api_key: "not-used",  // Copilot ignores x-api-key when Authorization is present
  base_url: "https://api.individual.githubcopilot.com",
  headers: {  // Must be object, NOT JSON.stringify()
    "Authorization": "Bearer <oauth-token>",
    "User-Agent": "GitHubCopilotChat/0.35.0",
    "X-Initiator": "user",
    "Openai-Intent": "conversation-edits",
    ...
  },
});
cr.setPrimary("PiClient");

const result = await runtime.callFunction("MyFunc", args, ctx, null, cr, [collector]);
const parsed = result.parsed(false);
```

### Pi Extension API (relevant subset)

```typescript
export default function(pi: ExtensionAPI) {
  pi.registerTool({ name, description, parameters, execute });
  pi.events.emit("pi-baml:ready", libraryObject);
  // No session_start handler needed — library is stateless.
  // Consumers pass ctx.modelRegistry on each library method call.
  pi.on("before_agent_start", (event) => {
    // Append <available_baml_functions> block to system prompt
  });
}
```

### Discovery Priority

Functions are discovered from these directories (lowest → highest priority):

| Priority | Path | Group prefix |
|----------|------|--------------|
| 1 (lowest) | Pi's resolved skill paths (lazy) | `skill:` |
| 2 | `~/.agents/baml/<group>/` | none |
| 3 | `~/.pi/baml/<group>/` | none |
| 4 | `[settings.functionsDirs]` | none |
| 5 | `<cwd>/.pi/baml/<group>/` | none |
| 6 (highest) | `<cwd>/.agents/baml/<group>/` | none |

`skill:` groups are discovered lazily on the first agent turn (via `before_agent_start`) using Pi's fully resolved skill paths — automatically includes profile-specific directories, npm-packaged skills, and explicit `--skill` flags. Two layouts are supported per skill:

1. `<skill>/baml/*.baml` — dedicated subdirectory (preferred)
2. `<skill>/*.baml` — flat files alongside SKILL.md

`skill:` groups are available for `baml_run` but excluded from the system prompt injection.

### System Prompt Injection

`src/index.ts` registers a `before_agent_start` handler that appends an `<available_baml_functions>` XML block to the system prompt — mirroring how `<available_skills>` works. Only non-`skill:` groups with descriptions appear in this block. The agent uses it to decide when to call `baml_list` for full signatures.

Disable with `baml.systemPrompt: false` in settings.

### Pi API type → BAML provider mapping

| Pi `model.api` | BAML `provider` | Status |
|----------------|-----------------|--------|
| `anthropic-messages` | `anthropic` | ✅ Supported |
| `openai-completions` | `openai-generic` | ✅ Supported |
| `google-generative-ai` | `google-ai` | ✅ Supported |
| `google-vertex` | `vertex-ai` | ✅ Supported |
| `bedrock-converse-stream` | `aws-bedrock` | ✅ Supported |
| `openai-responses` | — | ❌ Unsupported (BAML 0.85.0) |

## Conventions

- **ESM only** — no CommonJS
- **.baml files always use `client PiClient`** — model selection at call time
- **No `any`** in public API
- **Error handling:** explicit, wrap with context, never swallow
- **Testing:** table-driven, behavior-focused, mock at system boundaries
- **Naming:** camelCase functions, PascalCase types, kebab-case files

## Build & Test

```bash
npm install
npm run build          # tsup → dist/
npm run typecheck      # tsc --noEmit
npm run lint           # eslint src/ tests/
npm test               # vitest (unit tests)
npm run test:integration  # real BAML compilation
```

## Dependencies

| Package | Purpose |
|---------|---------|
| `@boundaryml/baml` | BAML runtime (native NAPI binary) |
| `@earendil-works/pi-coding-agent` | Pi types (dev only) |
| `typescript` | Build |
| `tsup` | Bundler |
| `vitest` | Test runner |
| `eslint` + `@typescript-eslint/*` | Linting |

## Important Constraints

1. **No network in factory** — all LLM calls happen at tool-invocation time
2. **Emit from factory** — `pi-baml:ready` MUST fire during factory, not session_start
3. **Soft-fail** — if settings are invalid or BAML can't load, emit `{ available: false }`
4. **Always setPrimary** — file-declared clients are ignored, tier model is always used
5. **Three tiers only** — light, standard, heavy. No arbitrary model strings.
6. **Headers must be objects** — BAML's `addLlmClient` options accept `{ [key: string]: any }`. Headers must be passed as a JS object, never `JSON.stringify()`.
7. **No `openai-responses` models** — BAML 0.85.0 lacks this provider. Throws explicit error.
8. **Copilot auth is provider-aware** — `anthropic` needs Bearer in headers; `openai-generic` uses `api_key` natively (see ADR-013).
9. **Enriched tool output** — `baml_run`/`baml_exec` return `{ result, model, tier }` envelope. The render layer unwraps this for display and shows model/tier in the footer.
10. **Discovery runs in two phases** — `discoverBamlGroups(cwd, functionsDirs, [])` scans standard dirs at factory time; skill-colocated BAML is discovered lazily on first `before_agent_start` using Pi's resolved skill paths.
11. **ModelRegistry is explicit** — library methods require `modelRegistry` as a parameter. No internal state, no session_start handler, no lifecycle coupling (see ADR-007).
12. **`skill:` prefix for skill-colocated BAML** — literal in registry key. Qualified names: `skill:diagnose/ClassifyBugPhase`.
13. **System prompt excludes skill groups** — only non-`skill:` groups appear in `<available_baml_functions>`.
14. **README.md filtered from compilation** — never passed to `BamlRuntime.fromFiles()`. Only `.baml` files are compiled.

---
> Source: [PedroKlein/pi-baml](https://github.com/PedroKlein/pi-baml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
