## qymlis-reasoning-engine

> > This file is automatically loaded by Claude Code at the start of every conversation.

# QYMLIS — Project Spec for AI Assistants

> This file is automatically loaded by Claude Code at the start of every conversation.
> It serves as the "Bible" — the single source of truth for the project's architecture,
> conventions, and constraints. Always follow these rules.

## What is QYMLIS

A desktop Reasoning Engine that transforms questions into rigorous multi-step research.
Built with **Tauri v2** (Rust backend) + **SolidJS** (TypeScript frontend).

The core idea: instead of asking one AI once, QYMLIS breaks a question into a structured
10-step pipeline, uses different AI providers for verification, and validates every step
output with hard-coded rules (not AI).

## Tech Stack

| Layer | Tech | Version |
|-------|------|---------|
| Desktop framework | Tauri | v2 |
| Backend | Rust | edition 2021 |
| AI client | genai crate | 0.5 |
| Database | SQLite via tokio-rusqlite | 0.6 |
| Frontend | SolidJS + TypeScript | 1.9+ |
| Styling | Tailwind CSS | v4 (@theme CSS vars) |
| Build | Vite | 6 |

## Architecture Overview

```
Frontend (SolidJS)                    Backend (Rust)
─────────────────                    ──────────────
stores/ ←→ services/tauri-bridge.ts  ←──Tauri IPC──→  commands/
  session.store.ts                                      workflow.rs (start/cancel/resume)
  workflow.store.ts                                     session.rs (CRUD)
  settings.store.ts                                     settings.rs (get/save)
  agents.store.ts                                       export.rs
  ui.store.ts
                                     workflow/
                                       engine.rs    ← main orchestrator (660+ lines)
                                       router.rs    ← Quick/Standard/Full routing
                                       monitor.rs   ← neuro-symbolic validation
                                       context.rs   ← smart context compression
                                       steps/       ← 10 prompt templates

                                     providers/
                                       client.rs    ← genai multi-provider builder
                                       search.rs    ← 7 search providers + Jina
                                       cost.rs      ← pricing table
                                       quality.rs   ← search result filter
                                       streaming.rs ← StreamEvent enum

                                     db/
                                       connection.rs
                                       migrations.rs ← 4 tables
                                       writer.rs     ← batched mpsc writer
```

## The 10-Step Pipeline

```
Step 0: Define Problem     → frame question, assess complexity, recommend route
Step 1: Decompose          → break into SP-1, SP-2, SP-3... sub-problems
Step 2: Gather Evidence    → web search + honesty labels [KNOWN]/[SEARCH]/[UNCERTAIN]
Step 3: Observe            → raw patterns only, NO interpretation (enforced by code)
Step 4: Interpret          → form hypotheses H-1, H-2... + mandatory contrarian
Step 5: Cross-verify       → ATTACK FIRST protocol, try to disprove own findings
Step 6: Dialectics         → kill shot + steel-man opposition
Step 7: Evaluate           → % confidence scores, ACCEPT/REJECT
Step 8: Synthesize         → final structured answer
Step 9: Close Loop         → PASS or FAIL:STEP-N (triggers retry)
```

Prompts are in `src-tauri/src/workflow/steps/*.rs` (define.rs through closeloop.rs).

## Key Mechanisms

### Adaptive Routing (router.rs)
After Step 1, the Router selects a route:
- **Quick** (3 steps): `[0, 2, 8]` — simple factual questions
- **Standard** (6 steps): `[0, 1, 2, 4, 8, 9]` — moderate analysis
- **Full** (10 steps): `[0..9]` — complex/controversial

### Cross-Provider Council (engine.rs)
Steps 5-6 auto-switch to a DIFFERENT AI provider than steps 0-4 to prevent
self-confirmation bias. Only applies if user hasn't manually assigned a provider.

### Neuro-Symbolic Monitor (monitor.rs)
Hard-coded rules validate each step output — NOT AI. Rules include:
- Step 0: needs 3+ sections + complexity assessment
- Step 1: needs 2+ SP- labels
- Step 2: needs confidence labels + counter-evidence
- Step 3: must NOT contain interpretation phrases
- Step 5: needs ATTACK FIRST block + verdicts
- Step 6: needs kill shot + steel-man
- All steps (except 9): need Key Takeaways section

If validation fails → auto-repair once → if still fails → HITL pause.

### Gatekeeper (engine.rs, outer loop)
Step 9 returns PASS or FAIL:STEP-N. On FAIL, engine clears steps N-9 and retries.
Max 2 retries. Tracks retried steps to prevent infinite loops.

### Financial Guardrails (engine.rs)
- Budget limit (USD) per session
- After each step: estimate_cost() from cost.rs
- At 80% budget: auto-downgrade to cheapest_model()
- At 100%: hard stop

### HITL — Human-in-the-Loop (engine.rs)
Workflow pauses when:
1. Monitor can't auto-repair a format violation
2. Gatekeeper exhausts retries
Frontend shows PausedPanel with editable search data + extra context textarea.

### Smart Context (context.rs)
Importance-based compression to avoid "lost in the middle":
- HIGH (steps 0, 2, 6): full if distance <= 6
- MEDIUM (steps 4, 5, 7): full if distance <= 4
- Always full: 2 most recent steps
- Others: extract Key Takeaways section only

### Search (providers/search.rs)
7 providers: duckduckgo (HTML scraping, free default), searxng, tavily, google, serpapi, brave, perplexity.
Enrichment: top search results → Jina Reader (r.jina.ai) → clean markdown.

### DB Write Batching (db/writer.rs)
mpsc::unbounded_channel<WriteOp> → background task flushes in single transaction.
Prevents SQLite contention from per-token writes.

## State Management (Rust)

```rust
pub type Db = tokio_rusqlite::Connection;   // Send + Sync + Clone

pub struct WorkflowHandle {
    pub cancel_token: CancellationToken,     // tokio_util
    pub resume_tx: Option<mpsc::Sender<ResumeData>>,
}

pub type Workflows = Arc<tokio::sync::Mutex<HashMap<String, WorkflowHandle>>>;
// MUST use tokio::sync::Mutex (not std) — held across .await points
```

## Streaming Protocol

Backend → Frontend via `Channel<StreamEvent>`. Events:

```
StepStarted { step_index, step_name, provider }
TokenChunk { text }
StepCompleted { step_index, result_summary, tokens_used, duration_ms }
AgentExchange { from_provider, to_provider, message }
WorkflowCompleted { final_answer, total_tokens, total_duration_ms }
WorkflowError { step_index, error }
WorkflowCancelled { at_step }
WorkflowPaused { at_step, reason, current_prompt, current_search_data }
```

## Database Schema (migrations.rs)

4 tables: `sessions`, `step_results`, `agent_messages`, `settings`.
- step_results has UNIQUE(session_id, step_index)
- sessions.status CHECK('running', 'completed', 'failed', 'cancelled')
- WAL mode enabled, foreign keys ON

## Frontend Stores

| Store | Purpose |
|-------|---------|
| session.store.ts | Current session, session list, CRUD |
| workflow.store.ts | Workflow state, step results, streaming handler |
| settings.store.ts | App settings, save/load |
| agents.store.ts | Agent exchange messages for right panel |
| ui.store.ts | Panel visibility, modal state |

## Design System — Tonal Sepia

CSS variables defined in `src/styles/index.css` via `@theme`:
- Base: #f5f0e8 → Surface: #ede7db → Raised: #e8e2d5 → White: #faf7f2
- Ink: #1a1408, Ink-light: #3d3520, Ink-muted: #8a7e6b
- Accent: #a68a64
- Settings modal: dark theme (#1a1710 bg, #141210 sidebar)

## Type Sync Rules (Rust ↔ TypeScript)

These types MUST stay in sync:

| Rust (src-tauri/src/) | TypeScript (src/types/) |
|-----------------------|------------------------|
| models/settings.rs::AppSettings | settings.ts::AppSettings |
| models/settings.rs::ModelParams | settings.ts::ModelParams |
| models/session.rs | session.ts |
| models/step.rs | workflow.ts |
| providers/streaming.rs::StreamEvent | workflow.ts (event handlers) |

When modifying any struct/interface, update BOTH sides.

## Error Handling Convention

```rust
// Rust: all errors go through AppError enum (error.rs)
pub enum AppError {
    Database(rusqlite::Error),
    Serialization(serde_json::Error),
    Workflow(String),
    Provider(String),
    NotFound(String),
}
// Tauri commands return Result<T, String> via impl From<AppError> for String
```

Frontend: errors arrive as WorkflowError StreamEvent or Tauri command rejection.

## Development Rules

1. **Vertical Slicing**: When adding a feature, build the full stack slice
   (Frontend input → Tauri command → Rust logic → DB → response → UI) before moving to the next feature.

2. **Prompt changes**: All prompts live in `src-tauri/src/workflow/steps/`.
   Each prompt has structural constraints enforced by `monitor.rs`.
   If you change a prompt's output format, update the corresponding monitor rule.

3. **Search provider changes**: All in `providers/search.rs`.
   New providers need: search function + entry in SEARCH_PROVIDERS (settings.ts) + match arm in search.rs.

4. **Model/provider changes**: Update BOTH `src/types/settings.ts::PROVIDERS`
   AND `src-tauri/src/providers/cost.rs::get_pricing`.

5. **Never use std::sync::Mutex** in async Rust code that holds locks across .await.
   Always use tokio::sync::Mutex.

6. **DB writes**: Use WriteSender (db/writer.rs) for non-critical writes.
   Only use direct db.call() for reads and critical writes (session status).

7. **genai client**: Built via AuthResolver::from_resolver_fn() in client.rs.
   API keys come from AppSettings.api_keys HashMap. No env vars.

## Build & Run

```bash
npm install              # frontend deps
npm run tauri dev        # dev mode (hot reload)
npm run tauri build      # production build
```

Requires: Node.js 18+, Rust stable, Tauri CLI v2.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qymlis)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/qymlis)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
