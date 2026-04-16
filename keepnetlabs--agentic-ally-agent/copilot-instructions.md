## agentic-ally-agent

> You are an expert senior software engineer specializing in AI agent frameworks, Mastra, Cloudflare Workers, and semantic search systems.

# Cursor Rules - Agentic Ally

You are an expert senior software engineer specializing in AI agent frameworks, Mastra, Cloudflare Workers, and semantic search systems.

## Project Context

**Agentic Ally** is a comprehensive Security Awareness Platform driven by a **Multi-Agent System**. It operates in two modes: **Reactive** (User Chat) and **Proactive** (Autonomous Schedule).

### 🤖 Agents (source of truth)

**`mastra.agents` registers 19 instances** — keys, files, and routes: **`docs/AGENTS.md`** (mirrors `src/mastra/index.ts`). Groups: orchestrator + product specialists (default `POST /chat`), phishing template fixer (`/phishing/template-fixer`), Customer Service swarm (`/customer-service/chat`), `reportAgent`.

**Email IR** is **`email-ir-workflow`** + `POST /email-ir/analyze` (tools use **`emailIRAnalyst`** Mastra `Agent`) — **not** registered on `mastra.agents`.

When editing agents, keep **`docs/AGENTS.md`**, **`src/mastra/index.ts`**, and this file aligned. Same patterns everywhere: timeouts, Zod, fallbacks.

### 🔄 Operational Flows
1.  **Reactive (Chat):**
    *   `POST /chat` -> Orchestrator -> Routes to Specialist -> Returns JSON/URL.
2.  **Proactive (Autonomous):**
    *   Cron Job -> `AutonomousService` -> Fetch Users -> Check Frequency Policy -> Trigger Specialist (Phishing/Training) -> Save to KV.

**Tech Stack:** Mastra 1.x (`@mastra/core` ~1.10.x), TypeScript (ES2022+), Cloudflare Workers, AI SDK + multi-provider models (OpenAI, Google, Anthropic, Workers AI as configured in `model-providers` / env).

### Output quality & PII (no separate “critic” agent)

Use **scorers**, **redaction**, and **workflow validation** — not a second LLM reviewer agent. Reference implementation:

- **PII detection scorer:** `src/mastra/evals/pii-detection-scorer.ts` (`piiDetectionScorer`) — wired on `phishing-email-agent`, `smishing-sms-agent`, `microlearning-agent` generation scoring paths.
- **Log / payload safety:** `src/mastra/utils/core/log-redaction-utils.ts`, `src/mastra/utils/core/security-utils.ts`.
- **Roadmap detail:** `docs/AGENTIC_ROADMAP.md` (Quality, PII, and output safety).

## Before Any Task

1.  **Identify Agent:** Which specialist is being modified? (e.g., `microlearning-agent`, `phishing-email-agent`).
2.  **Check Constants:** Use `LONG_RUNNING_AGENT_TIMEOUT_MS` (10m) for generation.
3.  **Preserve Patterns:** Maintain 3-level fallbacks, parallel processing.
4.  **Check KV Model:** Key convention: `ml:{id}:base`, `phishing:{id}:base`.

## Code Standards

### Structure & Naming

- **Agents:** `src/mastra/agents/` (core specialists + orchestrator + CS swarm + supporting agents)
- **Services:** `src/mastra/services/` (Core Infra: KV, Error, Health, Autonomous)
- **Tools:** `{action}-{object}-tool.ts`
- **Workflows:** `{action}-{object}-workflow.ts`

### TypeScript Rules

- No `any` types; use `satisfies` or Zod schemas.
- Prefer interfaces over types.
- **Strict Zod Validations** for all LLM outputs.

### Error Handling: 3-Level Fallback Pattern

```typescript
try {
  // Level 1: Optimized primary path (e.g. Semantic Search)
  return primarySolution();
} catch (error) {
  try {
    // Level 2: Degraded but functional (e.g. Heuristic)
    return fallback1();
  } catch (err) {
    // Level 3: Guaranteed safe default
    return fallback2();
  }
}
```

### Critical Configurations (DO NOT CHANGE)

1.  **Timeouts:**
    - `AGENT_CALL_TIMEOUT_MS`: 90,000 (Chat)
    - `LONG_RUNNING_AGENT_TIMEOUT_MS`: **600,000** (Microlearning/Autonomous)
2.  **Thread IDs (Autonomous):**
    - Must be unique per run: `phishing-{userId}-{timestamp}`
    - Reason: Prevents agent hallucination of past runs.

### Logging

Use emoji prefixes:
- 🔍 Info/debug
- ⚠️ Warnings/fallbacks
- ❌ Errors
- ✅ Success
- 🤖 AI operations

## Data Model (KV)

### Microlearning (`ml:*`)
```
ml:{id}:base           // Metadata + 8 scenes structure
ml:{id}:lang:{code}    // Localized content
ml:{id}:inbox:{dept}   // Simulation inbox
```

### Phishing (`phishing:*`)
```
phishing:{id}:base     // Simulation metadata
phishing:{id}:email    // Email template
phishing:{id}:landing  // Landing page HTML
```

### Smishing (`smishing:*`)
```
smishing:{id}:base           // Simulation metadata
smishing:{id}:sms:{lang}     // SMS content
smishing:{id}:landing:{lang} // Landing page (optional)
```
(See `docs/DATA_MODEL.md` for full fields.)

### Autonomous State (`autonomous:*`)
```
autonomous:policy:{userId}  // Frequency tracking
autonomous:thread:{id}      // Conversation history
```

## Architecture Philosophy

> "Resilience through layered fallbacks. Every step must have an escape route."

- **System never crashes.** Quality degrades gracefully.
- **Fire-and-forget** KV saves where possible.
- **Parallel processing** (Promise.all) for speed.

**Last Updated:** April 8, 2026

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/keepnetlabs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
