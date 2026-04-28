## psycheros-alpha

> Persistent AI entity harness daemon built on Deno 2.x. Web chat UI with streaming LLM, tool execution, hierarchical memory, RAG, knowledge graph, lorebook, data vault, and identity management. Runs on port 3000.

# Psycheros — Agent System Card

Persistent AI entity harness daemon built on Deno 2.x. Web chat UI with streaming LLM, tool execution, hierarchical memory, RAG, knowledge graph, lorebook, data vault, and identity management. Runs on port 3000.

Psycheros is an **embodiment** — an interface through which the AI entity exists and interacts. The entity's canonical identity lives in [entity-core](https://github.com/zarilewis/entity-core) (MCP server). Psycheros syncs with it when `PSYCHEROS_MCP_ENABLED=true`.

## First-Person Convention

All prompts, system messages, tool descriptions, and code comments use the entity's first-person perspective ("I am...", "I should..."), never second-person. The entity internalizes the system as *theirs*, not as rules imposed on them. See [docs/entity-philosophy.md](docs/entity-philosophy.md) for the full rationale. **Maintain this convention in all contributions.**

## Commands

```bash
deno task dev          # Development with hot reload
deno task start        # Production mode
deno task stop         # Graceful shutdown
deno check src/main.ts # Type check
deno lint              # Lint
```

## Setup

```bash
cp .env.example .env   # Then set LLM API key and PSYCHEROS_TOOLS
```

LLM connections are configured via **Settings > LLM Settings** in the web UI. Multiple named connection profiles can be created (OpenRouter, OpenAI, Alibaba/Qwen, NanoGPT, or custom endpoints). One profile is marked active for chat. On first run, a default profile is created from `ZAI_*` environment variables if set.

### With MCP (entity-core)

```bash
PSYCHEROS_MCP_ENABLED=true deno task dev
```

## Key Files

| File | Purpose |
|------|---------|
| `src/main.ts` | Entry point, MCP initialization |
| `src/types.ts` | Shared types (SSEEvent, LLMContextSnapshot, ToolCall) |
| `src/entity/loop.ts` | Agentic loop — LLM calls, tool execution, context capture, image/tool-arg fading |
| `src/entity/context.ts` | Context loading (supports MCP client) |
| `src/server/routes.ts` | API endpoints and handlers |
| `src/server/state-changes.ts` | Unified state mutations |
| `src/server/broadcaster.ts` | Persistent SSE for background updates |
| `src/tools/registry.ts` | Tool catalog (`AVAILABLE_TOOLS`) and registration |
| `src/tools/tools-settings.ts` | Tool enable/disable persistence and resolution |
| `src/tools/custom-loader.ts` | Dynamic loader for user-written tools in `custom-tools/` |
| `src/tools/web-search.ts` | Web search tool (Tavily / Brave) |
| `src/tools/send-discord-dm.ts` | Discord DM tool (sends DMs via Discord bot API) |
| `src/tools/control-device.ts` | Home automation tool (smart plug control via Shelly API) |
| `src/tools/generate-image.ts` | Image generation tool (OpenRouter, Gemini), auto-captioning (dual short/long) |
| `src/tools/describe-image.ts` | Image captioning tool (Gemini, OpenRouter), shared caption logic (dual short/long) |
| `src/tools/look-closer.ts` | Re-examine images for detailed descriptions after context fade |
| `src/llm/provider-presets.ts` | LLM provider types, connection profile types, and default presets |
| `src/llm/discord-settings.ts` | Discord settings type, load/save, token masking |
| `src/llm/home-settings.ts` | Home automation settings type, load/save (device list) |
| `src/llm/image-gen-settings.ts` | Image generator + captioning config type, load/save, masking |
| `src/llm/entity-core-settings.ts` | Entity-core LLM override settings type, load/save (model, temperature, maxTokens) |
| `src/tools/identity-helpers.ts` | Identity file utilities (XML parsing, MCP fallback, local snapshot restore) |
| `src/tools/identity-custom.ts` | Custom identity file tool (create, append, prepend, update_section, rewrite_section) |
| `src/memory/mod.ts` | Hierarchical memory system (daily summarization only; weekly/monthly/yearly consolidation moved to entity-core) |
| `src/rag/mod.ts` | RAG retrieval system |
| `src/mcp-client/mod.ts` | MCP client for entity-core connection |
| `src/lorebook/mod.ts` | Lorebook/world info system |
| `src/vault/mod.ts` | Data Vault — document storage and eager RAG |
| `src/db/schema.ts` | Database schema, migrations, vector table sync |
| `src/db/vector.ts` | sqlite-vec extension loading with auto-download from GitHub releases |
| `src/init/mod.ts` | Initialization — seeds identity, custom-tools, and vault template directories |
| `src/pulse/engine.ts` | Pulse system — autonomous scheduled entity prompts |
| `src/pulse/routes.ts` | Pulse API routes, CRUD, triggers, webhook endpoint |
| `src/pulse/templates.ts` | Pulse UI — settings hub card, editor, execution log |
| `src/pulse/timezone.ts` | Timezone conversion for local↔UTC Pulse scheduling |
| `src/tools/pulse-tools.ts` | Entity-facing Pulse tools (create, trigger, delete) |
| `src/push/mod.ts` | Push notification manager (VAPID keys, subscriptions, sending) |

## Situational Awareness

Real-time signal feeds injected into the entity's context every turn. Configured via Settings > Situational Awareness.

- **Current Conversation** — The conversation ID and title the entity is currently processing.
- **Last User Interaction** — Most recent human message across all threads (excludes Pulses). Entity sees timestamp (user's display timezone) and thread ID/title.
- **Device Detection** — Desktop or mobile, detected by frontend heuristic and sent with each `/api/chat` request.

Settings persist to `.psycheros/sa-settings.json`. Default `{ "enabled": true }`.

## LLM Connections

Psycheros supports multiple named LLM connection profiles. Each profile stores an API endpoint, key, model, worker model, and sampling parameters. One profile is marked **active** for chat. Profiles are managed via Settings > LLM Settings in the web UI (hub view with card grid, same pattern as Image Gen).

Supported provider presets: **OpenRouter** (default), OpenAI, Alibaba/Qwen, NanoGPT, Custom Endpoint. The `LLMClient` works with any OpenAI-compatible endpoint.

- Settings persist to `.psycheros/llm-settings.json` as `LLMProfileSettings` (array of `LLMConnectionProfile` + `activeProfileId`)
- Automatic migration from legacy flat `LLMSettings` format
- Entity-core LLM credentials are derived from the active profile on startup and dynamically updated when the active profile changes (triggers entity-core restart if connected)
- Entity-core LLM model/temperature/maxTokens can be overridden independently via Settings > Entity Core > LLM (persists to `.psycheros/entity-core-llm-settings.json`)
- Worker model (auto-titling, summarization) uses the profile's `workerModel` with thinking disabled

## External Connections

Psycheros supports third-party integrations organized under three tabs in Settings > External Connections:

### Channels

- **Discord DM** — Entity sends DMs via a Discord bot. Configured via Settings > External Connections > Channels or env vars (`DISCORD_BOT_TOKEN`, `DISCORD_DEFAULT_CHANNEL_ID`). Auto-enables the `send_discord_dm` tool when configured. Settings persist to `.psycheros/discord-settings.json`.

### Web Search

- **Provider Selection** — Choose None, Tavily, or Brave Search. API keys configured per provider. Auto-enables the `web_search` tool when a provider is set. Settings persist to `.psycheros/web-search-settings.json`.

### Home

- **Smart Devices** — Entity controls smart plugs (Shelly Plug) via local HTTP API. Configured via Settings > External Connections > Home. Auto-enables the `control_device` tool when at least one device is enabled. Settings persist to `.psycheros/home-settings.json`.

## Vision
Image generation and visual analysis configured via Settings > Vision (top-level settings card with Generators, Anchors, and Gallery tabs).

- **Image Generation** — Entity generates images via OpenRouter or Google Gemini. Supports multiple generator slots, anchor images for style/character reference, user image attachments, reference-based iteration (`input_image_path`), and auto-captioning of generated images. Auto-enables the `generate_image` tool when at least one generator is enabled. Settings persist to `.psycheros/image-gen-settings.json`. Generated images saved to `.psycheros/generated-images/`.
- **Image Captioning** — Auto-captions chat attachments and generated images via a configurable vision model (Gemini or OpenRouter). Captions produce both shortform (under 15 words) and longform descriptions. Longform fades to shortform in context after 5 turns; `look_closer` tool retrieves full details. Also provides the `describe_image` tool. Configured under Settings > Vision > Generators tab. Auto-enables `describe_image` and `look_closer` when a captioning provider is configured.
- **Gallery** — Browse all generated and user-uploaded images with thumbnails, category badges, metadata, copy-to-clipboard, and lightbox. Server-rendered on tab load with client-side pagination (24/page), derived from filesystem + messages (no DB table).

## Core Patterns

**Module structure**: Each `src/*/` has a `mod.ts` barrel file. Import from `mod.ts`, not internal files.

**Adding a built-in tool**:
1. Create `src/tools/my-tool.ts` implementing the `Tool` interface
2. Register in `AVAILABLE_TOOLS` in `src/tools/registry.ts`
3. Add tool name to the appropriate category in `TOOL_CATEGORIES` in `src/tools/tools-settings.ts`
4. For auto-enablement: add to `autoEnabled` array in `src/server/server.ts`
5. For UI updates: use state-change function, return `affectedRegions`

**Adding a custom tool** (no core code changes needed):
1. Create `custom-tools/my-tool.js` exporting a default `Tool` object
2. Or use the **Import Tool** button on Settings > Tools > Custom tab to upload from the UI
3. Toggle it on to enable

**State changes** (for reactive UI):
1. Add function to `src/server/state-changes.ts` returning `{ success, data, affectedRegions }`
2. Synchronous: return from tool (flows through chat stream)
3. Background: call `getBroadcaster().broadcastUpdates()` directly

**SSE channels**:
- `POST /api/chat` — per-request stream (message_id, context, thinking, content, tool calls, metrics, done). Client drains stream on conversation switch so server persists full response; explicit Stop still aborts.
- `POST /api/chat/retry` — same SSE format, re-attempts last user message without re-persisting it. Used by the Retry button shown when a turn fails with no assistant content.
- `GET /api/events` — persistent channel (dom_update events, Pulse streaming: content, thinking, tool_call, tool_result, done, message_id)

**Tool execution concurrency**: `ToolRegistry.executeAll()` uses a promise mutex to serialize tool execution across concurrent turns, preventing race conditions on shared resources (identity files, knowledge graph, memories).

**User data protection**:
- `identity/`, `memories/`, `.snapshots/`, `data/vault/` are **runtime-only directories** — gitignored, never committed
- To change identity defaults, edit `templates/identity/` (committed). `src/init/mod.ts` seeds `identity/` from templates on first run if empty. Vault documents in `templates/vault/` are seeded into the global Data Vault on first startup. **Never `git add` files from `identity/`** — they contain user-specific entity data.
- Entity-core is canonical source; local `identity/` is a cache when MCP is enabled

## Documentation Index

| Document | Purpose |
|----------|---------|
| [docs/entity-philosophy.md](docs/entity-philosophy.md) | First-person convention rationale, ownership, embodiment concept |
| [docs/configuration.md](docs/configuration.md) | All env vars, available tools, RAG/MCP settings, migration commands |
| [docs/tools-reference.md](docs/tools-reference.md) | Tool system, identity tiers, MCP fallback, core prompt file structure |
| [docs/memory-and-rag.md](docs/memory-and-rag.md) | Memory hierarchy, consolidation, 4 RAG systems (memory, chat, lorebook, vault), vector search |
| [docs/ui-features.md](docs/ui-features.md) | Context viewer, stop generation, retry failed turn, message editing, appearance, situational awareness, graph viz |
| [docs/api-reference.md](docs/api-reference.md) | Full API endpoints, dual SSE architecture, retry stream |
| [docs/code-review-findings.md](docs/code-review-findings.md) | Code review bugs fixed, architectural decisions |
| [docs/security-audit.md](docs/security-audit.md) | Security audit findings, threat model, accepted risks |
| [docs/deployment/docker-strategy.md](docs/deployment/docker-strategy.md) | Dockerfile design, volumes, env vars, Deno caching |
| [docs/deployment/ci-cd.md](docs/deployment/ci-cd.md) | GitHub Actions workflow, GHCR, build pipeline |
| [docs/deployment/unraid-setup.md](docs/deployment/unraid-setup.md) | UnRAID container config, Authelia, reverse proxy |

## Documentation System

This project uses a 4-layer documentation architecture. Each layer has a distinct purpose — no layer should duplicate information that belongs in another.

### Layers

1. **CLAUDE.md** (this file) — Agent system card. How to operate in this repo. Index to everything else. Target ≤200 lines.
2. **README.md** — Architecture map. Component relationships, directory structure. The structural brain.
3. **docs/** — Deep reference articles. One topic per file. Living documents updated when their subject changes.
4. **Claude Code auto-memory** (`~/.claude/projects/`) — Ephemeral, machine-local state. Session context, local env details, in-progress work. Never committed.

### When to Update

| Trigger | CLAUDE.md | README.md | docs/ | Auto-memory |
|---------|-----------|-----------|-------|-------------|
| New tool/feature added | Update key files if needed | Update architecture if structural | Update relevant doc | — |
| Architecture change | Update if operations change | Update affected sections | Update affected docs | — |
| Bug fix / minor change | No | No | Update if doc covers it | — |
| Environment change | No | No | No | Yes |
| Pre-commit (significant) | Verify index accuracy | Sweep for staleness | Verify touched topics | — |

### Pre-Commit Sweep

Before significant commits:
1. Verify this index table is accurate and complete
2. Confirm README.md reflects current architecture
3. Check that docs/ articles affected by code changes are still accurate
4. Ensure no committed file contains ephemeral state (IPs, paths, session context)
5. Confirm this file is ≤200 lines

### Ephemeral vs. Committed

**The portability test:** If someone cloned this repo fresh, would this information help them? If **yes** → committed docs. If **no** → auto-memory.

- Committed: architecture, tool reference, conventions, configuration, API routes
- Ephemeral: local paths, API keys, current branch, test database state, session progress

## Related Projects

- [entity-core](https://github.com/zarilewis/entity-core) — MCP server holding the entity's canonical identity and memories

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zarilewis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
