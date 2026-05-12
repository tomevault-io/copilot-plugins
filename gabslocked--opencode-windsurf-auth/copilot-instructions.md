## opencode-windsurf-auth

> Guidance for AI agents working on this repository.

# AGENTS.md

Guidance for AI agents working on this repository.

## Overview

OpenCode plugin that exposes Windsurf/Codeium models as an OpenAI-compatible local API. Starts an HTTP server on `127.0.0.1:42100` providing `/v1/chat/completions` (streaming SSE) and `/v1/models`. Supports 90+ models including `swe-1.5`, `claude-4.5-opus-thinking`, `gpt-5.2`, `gemini-3.0-pro`, and more.

**Key insight**: Windsurf does NOT use REST APIs or cloud OAuth — it runs a local `language_server_macos` gRPC server. This plugin discovers credentials from the running process and translates REST ↔ gRPC.

## Build & Test

```bash
bun install           # install dependencies
bun run build         # compile TypeScript (tsconfig.build.json)
bun run typecheck     # type check only (tsconfig.json)
bun test              # run unit tests
bun test tests/unit   # unit tests only
```

**Runtime**: Requires **Bun** (uses `Bun.serve()`). Will not work under Node.js.

## Architecture

```
Client (OpenAI-format)
  → POST localhost:42100/v1/chat/completions
  → plugin.ts (Bun HTTP server)
    → auth.ts: discover CSRF, port, API key from running Windsurf
    → models.ts: resolve model name → protobuf enum value
    → discovery.ts: parse extension.js for protobuf field numbers
    → grpc-client.ts: encode protobuf, HTTP/2 POST to localhost:{port}
      → /exa.language_server_pb.LanguageServerService/RawGetChatMessage
  → Windsurf language server (local gRPC)
  → Windsurf cloud inference
  → Streaming protobuf response
  → grpc-client.ts: decode protobuf, yield text chunks
  → plugin.ts: wrap as SSE `data: {...}\n\n` chunks
  → Client receives OpenAI-format stream
```

## Module Structure

```
index.ts                    # Package entry, re-exports plugin
src/
├── plugin.ts               # Main: Bun HTTP server, OpenAI-compat endpoints,
│                           #   tool-call planning, streaming/non-streaming responses
├── constants.ts            # API endpoints, gRPC service paths, model IDs,
│                           #   keychain constants, rate limit config
└── plugin/
    ├── auth.ts             # Credential discovery: ps aux → CSRF, lsof → port,
    │                       #   sqlite3 state.vscdb → API key, process args → version
    ├── discovery.ts        # Dynamic protobuf field number discovery from extension.js
    │                       #   (survives Windsurf version updates)
    ├── grpc-client.ts      # HTTP/2 gRPC client with hand-rolled protobuf encoding/decoding
    │                       #   (no protobuf library), async generator for streaming
    ├── models.ts           # Model name → enum mappings (90+), variant system,
    │                       #   alias resolution, reverse enum→name mapping
    └── types.ts            # ModelEnum numeric values (extracted from extension.js),
                            #   ChatMessageSource enum, TypeScript interfaces

tests/
├── README.md               # Test documentation
├── unit/
│   └── variant.test.ts     # Model variant resolution tests
└── live/
    ├── request.ts          # Send test requests to running Windsurf
    ├── test-all-models.ts  # Test all model enum values
    ├── test-complete.ts    # Completion integration test
    ├── verify-plugin.ts    # Plugin verification
    ├── analyze.ts          # Analyze captured traffic
    ├── capture.sh          # tcpdump capture script (requires sudo)
    └── ngrep-capture.ts    # ngrep-based capture
    └── model-test-results.json  # Saved test results

docs/
├── WINDSURF_API_SPEC.md    # gRPC API specification (protobuf schemas)
└── REVERSE_ENGINEERING.md  # How credentials and protocol were discovered
```

## Credential Discovery (auth.ts)

| Credential | Source | Method |
|------------|--------|--------|
| CSRF token | Process args | `ps aux \| grep language_server_macos` → regex `--csrf_token ([a-f0-9-]+)` |
| gRPC port | Process sockets | Extract PID → `lsof -p PID -i -P -n \| grep LISTEN` → first port after `extension_server_port`. Fallback: `extension_server_port + 3` |
| API key | VSCode state DB | `sqlite3 ~/Library/Application Support/Windsurf/User/globalStorage/state.vscdb "SELECT value FROM ItemTable WHERE key = 'windsurfAuthStatus'"` → parse JSON → `.apiKey`. Fallback: `~/.codeium/config.json` |
| Version | Process args | regex `--windsurf_version ([^\s]+)` from process args. Fallback: `1.13.104` |

**Platform paths for state.vscdb**:
- macOS: `~/Library/Application Support/Windsurf/User/globalStorage/state.vscdb`
- Linux: `~/.config/Windsurf/User/globalStorage/state.vscdb`
- Windows: `%APPDATA%/Windsurf/User/globalStorage/state.vscdb`

## gRPC Protocol

### Endpoint
```
POST http://localhost:{port}/exa.language_server_pb.LanguageServerService/RawGetChatMessage
Headers:
  content-type: application/grpc
  te: trailers
  x-codeium-csrf-token: {csrf_token}
```

### Request Format (RawGetChatMessageRequest)
Hand-encoded protobuf (no library):
- Field 1: `Metadata` message (api_key, ide_name, ide_version, extension_version, session_id, locale)
- Field 2: `ChatMessage` repeated (message_id, source enum, timestamp, conversation_id, intent/text)
- Field 3: `system_prompt_override` string (if system message present)
- Field 4: `chat_model` varint (model enum value)
- Field 5: `chat_model_name` string (optional, for variant fidelity)

### Response Format (RawGetChatMessageResponse)
gRPC-framed protobuf:
- Field 1: `delta_message` (RawChatMessage)
  - Field 5: `text` string (the content we extract)

### Message Encoding Rules
- User/system/tool messages: wrapped in `ChatMessageIntent` → `IntentGeneric` (field 5 as nested message)
- Assistant messages: plain text in field 5 (no intent wrapper)
- Source enum: USER=1, SYSTEM=2, ASSISTANT=3, TOOL=4

## Dynamic Field Discovery (discovery.ts)

Protobuf field numbers can change between Windsurf versions. `discovery.ts` parses the installed `extension.js` to find the current `Metadata` message field mapping:

```
/Applications/Windsurf.app/Contents/Resources/app/extensions/windsurf/dist/extension.js
```

It searches for `newFieldList(()=>[...])` patterns containing `"api_key"` and `"ide_name"` (excluding telemetry messages with `"event_name"`), then extracts `{no:X,name:"field_name"}` pairs.

Default fallback if discovery fails:
```typescript
{ api_key: 1, ide_name: 2, ide_version: 3, extension_version: 4, session_id: 5, locale: 6 }
```

## Model System (models.ts + types.ts)

### Enum Values
Extracted from Windsurf's `extension.js`. Use the extraction script to detect missing models:
```bash
bun run extract-models           # report missing models
bun run extract-models --update  # auto-patch types.ts + models.ts
bun run extract-models --json    # machine-readable output
```

The script parses the `setEnumType(…,"exa.codeium_common_pb.Model",[…])` definition from extension.js, filters out internal/embedding/tab/preview models, and compares against `types.ts`.

After `--update`, manually:
1. Move auto-extracted enums into the correct sections in `types.ts`
2. Add `VARIANT_CATALOG` entries for models with low/medium/high/thinking tiers
3. Clean up friendly names in `MODEL_NAME_TO_ENUM` / `ENUM_TO_MODEL_NAME`
4. Run `bun run typecheck` to verify

Examples: `SWE_1_5 = 359`, `CLAUDE_4_5_OPUS = 391`, `GPT_5_2_MEDIUM = 401`

### Variant System
Models with performance tiers use a variant catalog:
```
gpt-5.2:high  →  GPT_5_2_HIGH (402)
gpt-5.2:low   →  GPT_5_2_LOW (400)
gpt-5.2       →  GPT_5_2_MEDIUM (401, default)
```

Variants are separated by `:` (OpenCode convention) or `-` suffix. Supported via `VARIANT_CATALOG` in models.ts.

### Model Routing Mechanisms

Two types of model identifiers exist:

1. **Enum-based** (traditional): Numeric protobuf enum value in gRPC field 4. Used by older models (Claude 3.x-4.5, GPT-4.x-5.2, Gemini 2.x-3.0, etc.)
2. **String-UID** (newer): Server-side routing via string model UID in gRPC field 5. Used by Claude 4.6, GPT-5.3-Codex, Gemini 3.1 Pro, Kimi K2.5, GLM-5, Minimax M2.5.

String-UID models set `defaultModelUid` / `modelUid` in the variant catalog. When `modelUid` is present, `resolveModel()` returns it and `buildChatRequest()` sends it in field 5.

Some models also use `MODEL_PRIVATE_*` enum placeholders (e.g., `MODEL_PRIVATE_12` = GPT-5.1) where the server dynamically assigns display names. These are treated as enum-based models.

### Resolution Order
1. Check `VARIANT_CATALOG` (canonical id + variant)
2. Check `ALIAS_TO_ID` mapping
3. Fallback to `MODEL_NAME_TO_ENUM` legacy map
4. Default: `claude-3.5-sonnet` (enum 166)

### Runtime Model Discovery

Use `--state-db` flag with the extract script to discover models from a running Windsurf instance:
```bash
bun run extract-models --state-db    # show runtime models from state.vscdb
```

This reads `userStatusProtoBinaryBase64` from `state.vscdb` → decodes protobuf → extracts display name + model UID pairs. Useful for finding string-UID models not present in extension.js.

### Supported Model Categories
| Category | Count | Examples |
|----------|-------|----------|
| SWE | 5 | `swe-1.5`, `swe-1.5:thinking/slow`, `swe-1.6`, `swe-1.6:fast` |
| Claude | 30+ | `claude-3.5-sonnet` through `claude-4.6-opus` (with thinking/1m/fast variants) |
| GPT | 50+ | `gpt-4o` through `gpt-5.3-codex` (with low/med/high/xhigh/fast variants) |
| O-series | 10 | `o3`, `o3-pro`, `o4-mini` (with low/high variants) |
| Gemini | 16+ | `gemini-2.0-flash` through `gemini-3.1-pro` (with minimal/low/medium/high variants) |
| Codex Mini | 3 | `codex-mini`, `codex-mini:low`, `codex-mini:high` |
| DeepSeek | 5 | `deepseek-v3`, `deepseek-v3-2`, `deepseek-r1`, `deepseek-r1-fast/slow` |
| Llama | 5 | `llama-3.1-8b/70b/405b`, `llama-3.3-70b`, `llama-3.3-70b-r1` |
| Qwen | 8 | `qwen-2.5-7b/32b/72b`, `qwen-3-235b`, `qwen-3-coder-480b` |
| Grok | 4 | `grok-2`, `grok-3`, `grok-3-mini`, `grok-code-fast` |
| Other | 12+ | `mistral-7b`, `kimi-k2`, `kimi-k2.5`, `glm-4.7`, `glm-5`, `minimax-m2.5` |

## Tool Calling

Tool calling is **prompt-based**, not native gRPC tool support:

1. When `tools` array is present in the request, `buildToolPrompt()` constructs a text prompt listing available tools with JSON schemas
2. The prompt instructs the model to respond with exactly one JSON object:
   - `{"action":"tool_call","tool_calls":[{"name":"...","arguments":{...}}]}` for tool invocations
   - `{"action":"final","content":"..."}` for direct answers
3. Response is parsed via `parseToolCallPlan()` (JSON extraction + `<tool_call>` tag fallback)
4. Converted to OpenAI `tool_calls` format in the response
5. Tool execution stays on the OpenCode side (MCP/tool registry)

**Important**: assistant and tool messages from conversation history are filtered out before sending to Windsurf (line 489 in plugin.ts). Only user and system messages are forwarded.

## HTTP Server (plugin.ts)

### Endpoints
| Path | Method | Description |
|------|--------|-------------|
| `/v1/chat/completions` | POST | OpenAI-compatible chat completions (streaming + non-streaming) |
| `/v1/models` | GET | List all available models with variants |
| `/health` | GET | Health check (reports if Windsurf is running) |

### Server Details
- Host: `127.0.0.1` (localhost only)
- Default port: `42100` (falls back to random if occupied)
- Runtime: Bun (`Bun.serve()`)
- Singleton: stored in `globalThis.__opencode_windsurf_proxy_server__`
- Idle timeout: 100 seconds (for long chat streams)

## Dependencies

| Package | Purpose |
|---------|---------|
| `@connectrpc/connect` | gRPC connect protocol (declared but encoding is manual) |
| `@bufbuild/protobuf` | Protobuf runtime types (declared but encoding is manual) |
| `proper-lockfile` | File locking for account storage |
| `xdg-basedir` | XDG directory resolution |
| `zod` | Schema validation |
| `@opencode-ai/plugin` (dev) | OpenCode plugin type definitions |
| `@types/bun` (dev) | Bun runtime types |

**Note**: Despite `@connectrpc/connect` and `@bufbuild/protobuf` being in dependencies, the actual gRPC encoding/decoding in `grpc-client.ts` is fully manual (hand-rolled varint/length-delimited encoding). These packages may be used by `constants.ts` type definitions or planned future use.

## Known Limitations

- **Bun-only** — uses `Bun.serve()`, will not work with Node.js
- **Windsurf must be running** — no standalone auth; piggybacks on the running IDE process
- **macOS-focused** — Linux/Windows credential paths exist but are untested
- **Tool calling is prompt-based** — may be unreliable for complex tool schemas
- **No conversation continuity** — generates a new `conversation_id` per request (no multi-turn via Cascade sessions)
- **No native tool execution** — tools are planned by Windsurf but executed by OpenCode
- **Filters conversation history** — assistant/tool messages are stripped before sending to gRPC

## Related Projects

- [AlexStrNik/windsurf-api](https://github.com/AlexStrNik/windsurf-api) — VSCode extension approach (intercepts `windsurf.initializeCascade`)
- [yuxinle1996/windsurf-grpc](https://github.com/yuxinle1996/windsurf-grpc) — Complete reverse-engineered protobuf definitions
- [NoeFabris/opencode-antigravity-auth](https://github.com/NoeFabris/opencode-antigravity-auth) — Same pattern for Google Antigravity IDE
- [shengmeixia/windsurf-api-proxy](https://github.com/shengmeixia/windsurf-api-proxy) — CDP-based approach (Chrome DevTools Protocol)

---
> Source: [gabslocked/opencode-windsurf-auth](https://github.com/gabslocked/opencode-windsurf-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
