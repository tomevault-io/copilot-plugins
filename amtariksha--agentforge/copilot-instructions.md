## agentforge

> Multi-tenant AI chat agent platform. TypeScript modular monolith with Fastify.

# AgentForge — CLAUDE.md

## What This Is
Multi-tenant AI chat agent platform. TypeScript modular monolith with Fastify.
Rebuild of old Python/LangChain chatagent.

## Architecture
- Modular monolith: single Fastify app, clean module folders (gateway/, orchestrator/, memory/, tools/, admin/).
- PostgreSQL + pgvector + Redis + BullMQ.
- No LangChain. No LangGraph. No framework. Build agent loop from scratch.

## Key Patterns (from Claude Code Leak)
- Static/dynamic prompt split at cache boundary. Static block FIRST for cache hits.
- Three-layer memory: index (~300 tokens, always loaded) → topic files (on-demand) → transcripts (search only).
- Skeptical memory: verify against backend before acting on stored data.
- Strict Write Discipline: update memory only after confirmed writes.
- MicroCompact → AutoCompact → FullCompact. Circuit breaker: max 3 retries.
- Read tools concurrent. Write tools serial. Destructive tools need HITL approval.

## Tenant API Gateway
- Internal gateway module at `src/tools/tenant-gateway/`
- Each tenant gets a folder with handler functions
- Tool executor calls these directly (no HTTP) when `backend_mapping.type === 'internal'`
- When tenant deploys their own API, change tool definition to `type: 'external'` with URL
- Each tenant folder has its own DB connection config — they may use different databases
- Never share database connections across tenants

## Rules
- Every DB query includes tenant_id. RLS enforced.
- Zod schemas on ALL external inputs.
- Drizzle ORM. No raw SQL except migrations.
- Secrets in env vars. Never hardcode.
- Structured JSON logs with trace_id, tenant_id, conversation_id.
- Never log PII. Mask phone numbers and names.
- Haiku for classification. Sonnet for standard. Opus for complex/angry only.
- Guardrails on BOTH input and output. Never skip output guardrails.
- No `any` types. TypeScript strict mode.

## Commands
- `npm run dev` — Start development server (tsx watch)
- `npm run seed` — Seed database from config/seeds/*.seed.json
- `npm run migrate` — Push schema to database (drizzle-kit push)
- `npm run build` — Compile TypeScript
- `npm run typecheck` — Type check without emitting

## Do NOT Build
- No LangChain/LangGraph.
- No ChromaDB (use pgvector).
- No conversation flows / visual builder.
- No A/B testing.
- No internal CRM/Kanban (use outbound webhooks to external CRM).
- No voice yet.
- No formal blueprint/template system (configure tools directly).

## Build (v4 additions)
- Generative UI: content blocks (src/ui/content-blocks.ts), tool `ui` passthrough,
  render_ui platform tool, channel renderers. Messages.content = { blocks: [...] }.
- Cost & billing: model_pricing (versioned, cache write/read tiers), split
  tokensCached → tokensCacheWrite/tokensCacheRead, billing_periods, invoices,
  ledger, budget alert notifications. llm_usage_logs stays the single source.
- Async operator runtime on BullMQ for accounting-reconciliation and
  marketing agents (approval-gated, no direct customer chat).
- Tool registry v2: versioning, shared templates + per-tenant overrides,
  per-tool rate limits. MCP transport optional, external-facing only.

---
> Source: [amtariksha/agentforge](https://github.com/amtariksha/agentforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
