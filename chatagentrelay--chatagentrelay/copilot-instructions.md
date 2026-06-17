## chatagentrelay

> This file helps AI coding assistants understand and work with the CAR codebase.

# AI Agent Guide for Chat Agent Relay (CAR)

This file helps AI coding assistants understand and work with the CAR codebase.

## Quick Start
- Runtime: Bun (not Node.js)
- Language: TypeScript (strict mode)
- Test: `bun test --recursive` (~692 tests)
- Typecheck: per-package `bunx tsc --noEmit`
- Lint: `bun run lint`
- Configuration: SQLite database (default `./car.db` via `CAR_DB_PATH`); set `CAR_ENCRYPTION_KEY` to encrypt tokens and API keys at rest
- From `packages/server`, register channels, agents, and routes with the `car` CLI, then run `car start` (see `docs/getting-started.md`). Runtime changes to channels and agents apply without restart where supported
- **CLI examples** — Channels: `slack`, `teams`, `discord`, `telegram`, `whatsapp`, `webchat`, `lark`, `dingtalk`. Agents: `car agent add my-agent --endpoint=https://...` (A2A protocol). CAR integrates with agent runtimes exclusively via the A2A standard.

## Architecture
Chat Agent Relay (CAR) is a middleware framework between chat platforms and AI agent runtimes. Every message flows through a 7-event pipeline chain in an append-only ledger. The `AgentAdapter` interface (A2A-aligned) is the primary agent-side boundary, supporting structured events, HITL, and artifacts. Additional canonical event types capture supplementary interactions and agent lifecycle signals.

## Package Map
| Package | Purpose | Key Interface |
|---------|---------|--------------|
| contract-harness | Schema validation + adapter types + optional lifecycle | ContractHarnessValidators, AgentAdapter, ChannelAdapter, Disconnectable |
| config-store | Config storage (ConfigStore interface) | ConfigStore, SqliteConfigStore, RouteEngine |
| event-ledger | Event storage | LedgerStore |
| channel-web-chat | WebChat adapter + streaming helper | ChannelAdapter, `buildWebChatStreaming()` |
| channel-slack | Slack adapter | ChannelAdapter |
| channel-discord | Discord adapter | ChannelAdapter |
| channel-telegram | Telegram Bot API adapter | ChannelAdapter |
| channel-teams | Teams adapter | ChannelAdapter |
| channel-whatsapp | WhatsApp adapter | ChannelAdapter |
| channel-lark | Lark / 飞书 adapter | ChannelAdapter |
| channel-dingtalk | DingTalk / 钉钉 adapter | ChannelAdapter |
| middleware | Policy + routing | MiddlewarePipeline |
| backend-a2a | A2A protocol adapter | AgentAdapter (native) |
| delivery | Message delivery (orchestrates `ChannelSender`) | DeliveryOrchestrator |
| pipeline | Orchestration | FirstExecutablePathPipeline |
| server | Runtime (multi-agent via route rules; hot-pluggable channels/agents) | `car` CLI + HTTP API |
| adapter-conformance | Test suite | testChannelAdapter, testAgentAdapter |

## Key Patterns
- Channel adapters implement `ChannelAdapter` (unified ingress + egress); agent runtimes connect via the A2A protocol (`backend-a2a`)
- Optional lifecycle: `Disconnectable` in `contract-harness` with `isDisconnectable()`; server registries use it for cleanup when stopping channels
- Server `ChannelRegistry` / `AgentRegistry` extend behavior with `registerFactory(type, factory)`; built-in types are wired in `channel-factories.ts` and `agent-factories.ts` without adapter imports inside the registry modules
- `ChannelAdapter.createSender(event)` derives delivery target from the event's `provider_extensions`
- `describeCapabilities()` declares what a channel or agent supports (agents include `streaming`, `multiTurn`, `resume`, HITL, cancel, artifacts); the pipeline respects `multiTurn` and `streaming` when building history and invoking the agent
- Adapters never throw - return Result types
- Events are immutable and append-only
- correlation_id links all events in a request
- causation_id links parent -> child events
- provider_extensions preserves platform-specific data
- `X-Tenant-ID` header scopes ledger queries when `tenant.isolation=true`

## When Adding New Code
1. Check docs/rfcs/ for relevant specifications
2. Ensure new events validate against JSON Schema
3. Run conformance tests for new adapters
4. For webhook-based channels, implement or reuse platform webhook signature verification (`contract-harness` helpers and channel-specific verifiers)
5. For new built-in channel or agent types, add a factory in `channel-factories.ts` or `agent-factories.ts` and register it with `registerFactory` (avoid embedding adapter imports in the registry class itself)
6. If an adapter holds connections, implement `Disconnectable` so the server can tear down cleanly
7. Update CHANGELOG.md for user-facing changes

---
> Source: [ChatAgentRelay/ChatAgentRelay](https://github.com/ChatAgentRelay/ChatAgentRelay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
