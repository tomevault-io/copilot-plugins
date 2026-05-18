## kuramei-anterior-para-historico

> > This file provides context for AI agents (Codex review, Claude Code) working on this codebase.

# Kuramei — Agent Context (Codex / AI Reviews)

> This file provides context for AI agents (Codex review, Claude Code) working on this codebase.

---

## Project Overview

**Kuramei** is a personal AI assistant via WhatsApp for the Brazilian market.
- User sends natural language message → system understands → acts → generates standalone page delivered as link
- **Phase 0 (private):** validation with 10–20 users. No scale yet.
- **Origin:** Adapted fork of Dikta project (B2B multi-tenant). Dikta is reference only — not a dependency.

---

## Architecture

```
WhatsApp User
     │
     ▼
Meta Webhook ──► webhook-handler (Lambda)
                      │
                      ▼ InvocationType: Event (async)
                 agent-processor (Lambda)  [Sprint 2: to build]
                      │
                      ├──► DynamoDB (session + events)
                      ├──► OpenRouter API (DeepSeek V3 primary)
                      └──► Cloudflare KV (UI specs)
                                │
                                ▼
                      ${KURAMEI_BASE_URL}/ui/{token}
                      (Cloudflare Worker — static renderer)
```

## Monorepo Structure

```
kuramei/
├── apps/
│   └── webhook-handler/     # Lambda entry point
└── packages/
    ├── tools/               # @kuramei/tools — ToolRegistry, LLMTool
    ├── events/              # @kuramei/events — KurameiEvent, DynamoDB store
    ├── conversation/        # @kuramei/conversation — session manager
    ├── whatsapp/            # @kuramei/whatsapp — parser + sender + signature
    ├── presence/            # @kuramei/presence — identity resolver
    ├── agent/               # @kuramei/agent — AgentClient + LLM providers
    └── sdk/                 # @kuramei/sdk — AppConfig, BuiltApp, UISpecTemplate
```

---

## Tech Stack & Constraints

| Layer | Technology | Constraints |
|-------|------------|-------------|
| Runtime | Node.js 20 + ESM | Strict ESM: `.js` extensions on all internal imports |
| Language | TypeScript 5.6 strict | No `any` without comment explaining why |
| Build | Turborepo + pnpm workspaces | Never use `npm install` directly |
| Database | DynamoDB single-table | Session key: `SESSION#kuramei#<phoneNumber>` |
| LLM primary | DeepSeek V3 via OpenRouter | OpenAI-compatible API |
| LLM fallback | Gemini Flash 1.5 via OpenRouter | |
| LLM alt | Claude Sonnet (ClaudeProvider) | |
| UI edge | Cloudflare Worker + KV | Worker serves static HTML — never executes AI-generated code |
| Auth | JWT (shared secret Lambda ↔ Worker) | AWS Secrets Manager + Wrangler |

---

## Critical Architecture Rules

### 1. LLM Never Generates HTML/JS
The LLM generates **JSON spec only** (e.g., `{ type: "map", query: "Campinas" }`).
The Renderer (static TypeScript) transforms spec → HTML. This is a security boundary.

```typescript
// CORRECT — LLM output
const spec = { type: "map", query: "Campinas" }

// WRONG — LLM should never generate this
const html = "<div>...</div>"
```

### 2. ESM Imports — .js Extensions Required
All internal imports must use `.js` extension even though source files are `.ts`.

```typescript
// CORRECT
import { ToolRegistry } from './registry.js'
import { KurameiEvent } from '@kuramei/events'

// WRONG
import { ToolRegistry } from './registry'
import { ToolRegistry } from './registry.ts'
```

### 3. TypeScript Strict — No Untyped `any`
```typescript
// CORRECT — with justification
// eslint-disable-next-line @typescript-eslint/no-explicit-any
const rawPayload = body as any // Meta webhook payload varies by message type

// WRONG — silent any
const data: any = response
```

### 4. Single-Tenant — Hardcoded Tenant
```typescript
// CORRECT
const KURAMEI_TENANT_ID = 'kuramei' // hardcoded, Phase 0

// WRONG — don't make it dynamic before needed
const tenantId = process.env.TENANT_ID
```

### 5. Session Keys — DynamoDB Pattern
```typescript
// CORRECT
`SESSION#kuramei#${phoneNumber}`

// WRONG
`SESSION#${tenantId}#${phoneNumber}` // dynamic tenant removed in Phase 0
```

---

## Closed Architectural Decisions (Do Not Reopen)

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Channel | WhatsApp (Meta Business API) | Ubiquitous in Brazil, zero onboarding |
| LLM | OpenRouter (DeepSeek primary) | Cost efficiency for Phase 0 |
| Invocation | `InvocationType: Event` (direct async) | No EventBridge in Phase 0 |
| UI edge | Cloudflare Worker + KV | Edge latency, short TTL |
| Multi-tenancy | Hardcoded `kuramei` | Don't add complexity before needed |
| Workflows | `@kuramei/workflows` cut in Phase 0 | Revisit Sprint 3 |

---

## Sprint Status

### Sprint 1 — DONE (`08c4c21`)
`pnpm build` → 8/8 packages compiling, no TypeScript errors.

### Sprint 2 — IN PROGRESS: Generative UI Engine
**Done criterion:** send "navegar para Campinas" → receive link that opens map in browser.

1. **Spec Schema** — freeze JSON spec format the LLM generates
2. **Renderer** — TypeScript that transforms spec → HTML (static, not AI-generated)
3. **Cloudflare Worker** — serves `${KURAMEI_BASE_URL}/ui/{token}`, validates JWT, renders
4. **KV + Token** — agent-processor writes spec to KV with TTL, generates JWT URL

---

## Review Checklist

When reviewing code, verify:

- [ ] **ESM imports:** all relative imports end with `.js`
- [ ] **TypeScript strict:** no `any` without inline comment explaining why
- [ ] **No new packages** without justification (check against existing pnpm-lock.yaml)
- [ ] **LLM generates JSON spec only** — never HTML, JS, or executable content
- [ ] **Session key pattern** consistent: `SESSION#kuramei#<phoneNumber>`
- [ ] **Tenant hardcoded** to `'kuramei'` — no dynamic tenant lookup
- [ ] **Error handling** only at system boundaries (external APIs, Lambda entry)
- [ ] **`pnpm build` passes** — no TypeScript errors across all 8 packages
- [ ] **No `@kuramei/workflows`** — not implemented in Phase 0
- [ ] **No EventBridge** — Lambda invokes Lambda directly in Phase 0

---

## Commands

```bash
pnpm build          # build all packages via Turborepo
pnpm lint           # ESLint across packages
pnpm typecheck      # TypeScript check across packages
pnpm test           # run tests (vitest)
```

---

## Environment Variables (not yet provisioned)

```env
# Lambda (AWS Secrets Manager)
OPENROUTER_API_KEY=
WHATSAPP_VERIFY_TOKEN=
WHATSAPP_APP_SECRET=
WHATSAPP_ACCESS_TOKEN=
DYNAMODB_TABLE=
KURAMEI_JWT_SECRET=

# Cloudflare Worker (Wrangler secrets)
KURAMEI_JWT_SECRET=    # same secret as Lambda
KV_NAMESPACE=
```

---
> Source: [aleparreira/kuramei-anterior-para-historico](https://github.com/aleparreira/kuramei-anterior-para-historico) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
