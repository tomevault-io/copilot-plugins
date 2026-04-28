## agentosv2

> AgentOS v2 is an AI agent orchestration platform. Ground-up Python rebuild.

# AgentOS v2

## Overview

AgentOS v2 is an AI agent orchestration platform. Ground-up Python rebuild.

- **Backend**: FastAPI + SQLAlchemy + asyncio — `src/engine/app/api/`
- **Frontend**: React + Vite + Tailwind — `src/engine/app/ui/`
- **Database**: PostgreSQL + pgvector — `src/engine/db/`
- **Conductor**: Claude Opus agent loop — primary interface (building Phase 1)
- **Cloud Execution**: AWS ECS Fargate (planned later)

## Documentation

**Product overview:**

| Doc | What it covers |
|-----|---------------|
| [docs/PRODUCT_MATRIX.md](docs/PRODUCT_MATRIX.md) | **Complete feature matrix** — every feature, status, design doc, milestone |

**Core docs** (read in order):

| Doc | What it covers |
|-----|---------------|
| [docs/MANIFESTO.md](docs/MANIFESTO.md) | WHY: business vision, product goals, competitive positioning, design principles |
| [docs/ROADMAP.md](docs/ROADMAP.md) | WHEN: milestones M1-M12, status, what's built, what's next |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | HOW: system design, package structure, DB schema, long-term 5-layer vision |
| [docs/conductor-architecture.md](docs/conductor-architecture.md) | Conductor agent: runtime, tools, services, memory, channels, workspaces |
| [docs/conductor-ui-architecture.md](docs/conductor-ui-architecture.md) | Conductor UI: layout, inline cards, expand pattern, component inventory |
| [docs/conductor-prompt-design.md](docs/conductor-prompt-design.md) | Conductor system prompt: identity, planning protocol, communication style, examples |
| [docs/ONBOARDING.md](docs/ONBOARDING.md) | Key files, running locally |

**Reference docs:**

| Doc | What it covers |
|-----|---------------|
| [docs/USE_CASES.md](docs/USE_CASES.md) | Use cases, three levels of automation, Slack hero example |
| [docs/DESIGN.md](docs/DESIGN.md) | Visual design system: colors, typography, spacing, components |
| [docs/PROVIDER_SPEC.md](docs/PROVIDER_SPEC.md) | Provider/capability protocol system, I/O types |
| [docs/WORKSPACE_PATTERNS.md](docs/WORKSPACE_PATTERNS.md) | How to ship work: parallel vs serial patterns, brief templates |
| [docs/VISION.md](docs/VISION.md) | Detailed Layer 2-5 designs (persistent agents, evolution, hierarchy, knowledge) |

**Feature designs:**

| Doc | What it covers |
|-----|---------------|
| [docs/auth-authority-design.md](docs/auth-authority-design.md) | Auth, multi-org, roles, authority profiles, escalation, approvals |
| [docs/integrations-design.md](docs/integrations-design.md) | Integrations: OAuth, API keys, MCP, databases, Composio, webhooks, multi-account |
| [docs/infrastructure-design.md](docs/infrastructure-design.md) | AWS infra: ECS Fargate, RDS, Redis, SQS, S3+CloudFront, CI/CD, monitoring |
| [docs/memory-rag-design.md](docs/memory-rag-design.md) | Memory/RAG: 4 memory types, multi-strategy retrieval, entity graph, outcome learning |
| [docs/documents-design.md](docs/documents-design.md) | Documents: git-native storage, BlockNote editor, indexing, agent tools, import |
| [docs/system-workflows-design.md](docs/system-workflows-design.md) | System workflows: agent capabilities as editable workflows, compiler to Python |
| [docs/data-views-design.md](docs/data-views-design.md) | Data views: compact cards + full tables per data type, shared DataTable component |
| [docs/data-explorer-design.md](docs/data-explorer-design.md) | NocoDB integration (DEFERRED — for advanced analytics only) |
| [docs/ui-framework-design.md](docs/ui-framework-design.md) | UI framework: shadcn/ui components, CSS variable theming, community themes |
| [docs/ui-pages-spec.md](docs/ui-pages-spec.md) | Page-by-page UI spec: every view, component, state, interaction |
| [docs/event-persistence-design.md](docs/event-persistence-design.md) | Event persistence, workflow checkpoints, SSE replay, Temporal migration path |
| [docs/DEVOPS.md](docs/DEVOPS.md) | DevOps: deploy, monitor, debug, CI/CD, AWS ops, secrets, troubleshooting |
| [docs/engine-gaps-analysis.md](docs/engine-gaps-analysis.md) | Engine gap analysis: 15 gaps prioritized P0-P2, fix sequencing |
| [docs/cc-supervision-design.md](docs/cc-supervision-design.md) | CC supervision: SupervisionConfig, review modes (BUILT) |
| [docs/map-collect-design.md](docs/map-collect-design.md) | MAP/COLLECT dynamic fan-out, TRANSFORM upgrade (BUILT) |
| [docs/media-store-architecture.md](docs/media-store-architecture.md) | S3 media store for universal HTTPS URLs (BUILT) |
| [README.md](README.md) | Quick start, feature overview, provider table |

## Reference: v1 Codebase

The original AgentOS is at `/Users/joelgreen/Development/Aiprojects/adalytics/`.
Reference for patterns and context. Do not copy code directly.
Key v1 files:
- `server/prisma/schema.prisma` — data model (agents, runs, tasks, credentials)
- `server/src/agent-runtime/` — executor patterns (local, queue, container)
- `server/src/routes/callbacks/` — Fargate callback pattern
- `client/src/components/ConductorChat.jsx` — Conductor chat UX
- `CLAUDE.md` — v1 project guide

## Dev Server

**Always use the dev-server script.** It auto-finds free ports so multiple CC instances don't collide.

```bash
./scripts/dev-server.sh          # start both servers on free ports
./scripts/dev-server.sh stop     # stop servers
./scripts/dev-server.sh status   # show running URLs and PIDs
cat .devserver                   # get API_URL, CLIENT_URL, ports
```

Parse the READY output or read `.devserver` for the URLs. **Never hardcode ports.**

**Workspace isolation:** The dev-server script auto-detects if the Python editable install (`pip install -e .`) points to a different directory (e.g., the original repo vs a Conductor workspace) and re-runs the install to fix it. It also auto-symlinks `.env` from the canonical repo (`/Users/joelgreen/Development/Aiprojects/agentosv2/.env`) if missing. This ensures each workspace runs its own code with the shared API keys.

If you need to run manually (not recommended):
```bash
DATABASE_URL="postgresql://joelgreen@localhost:5432/agentos_v2" python3 -m uvicorn engine.app.api.main:app --port 8000
cd src/engine/app/ui && npx vite --port 5173
```

## Tests

```bash
python3 -m pytest tests/ -v          # all tests
python3 -m pytest tests/ -x -q       # stop on first failure
```

## Quality Process (MANDATORY for all work)

Follow this process for every piece of work. No shortcuts.

### Per-Step Cycle (for each numbered step in any workspace brief)

1. **Read first.** Read the files you'll modify. Understand existing patterns before writing code.
2. **Write tests first.** Failing tests before implementation. Cover: failure case, happy path, edge cases, backward compatibility.
3. **Implement.** Minimum code to make tests pass. Follow existing patterns exactly.
4. **Run ALL tests.** `python3 -m pytest tests/ -x -q` — full suite, not just new tests. Zero regressions.
5. **Self-review.** Check your diff: dead code? unnecessary abstractions? type hints? modules <300 lines? matches codebase style?
6. **`/review`** — Fix every issue. Re-run tests. Repeat until clean.
7. **Commit.** One commit per step, clear message.

### End-of-Work Cycle (after all steps complete)

8. **Full test suite.** `python3 -m pytest tests/ -v` — verbose, confirm count, zero failures.
9. **Post-analysis.** `git diff main..HEAD` — proportional changes? no unintended files? no unused imports?
10. **`/qa`** — Full QA pass. Fix anything found.
11. **`/review`** — Final review of complete changeset. Fix cross-cutting issues.
12. **Iterate.** Fix → re-test → re-review until clean.
13. **`/ship`** — Create PR.

### Documentation (MANDATORY)

Keep docs up to date. Documentation that drifts from reality is worse than no documentation.

- **When you add or change a feature:** Update the relevant design doc. If you add a new node type, update ARCHITECTURE.md. If you change how memory works, update memory-rag-design.md. If you add an API route, update conductor-architecture.md.
- **When you complete a milestone:** Update ROADMAP.md status table. Mark the milestone done. Update "What's Built" if significant.
- **When you make an architectural decision:** Document it in the relevant design doc with rationale. Future agents and humans need to know WHY, not just WHAT.
- **When a doc becomes stale:** Fix it or flag it. Don't leave wrong information in docs — it will mislead future work.
- **When you create a new module or service:** Add it to ONBOARDING.md key files section and ARCHITECTURE.md package structure.
- **Doc updates are part of the PR.** Not a separate follow-up. Ship code + doc updates together.

## Architecture

```
src/engine/
  services/          — Platform services: memory, conversations, tasks, tools, ops, channels (building)
  conductor/         — Conductor agent: loop, prompt, tools (first consumer of services) (building)
  executor/          — GraphExecutor, CC subprocess, interactive sessions, events
  models/providers/  — 6 AI providers, 42 models, 8 capability protocols
  graph/             — WorkflowGraph types, validator, routing rules
  generator/         — Chat-based workflow generation (being absorbed by Conductor)
  app/api/           — FastAPI routes (conductor, execute, workflows, runs, settings, costs)
  app/ui/            — React frontend (Conductor chat, GraphViewer, ExecutionStream)
  db/                — SQLAlchemy models, schema, session
  memory/            — pgvector memory store (planned, will use services/memory.py)
  evolution/         — Workflow evolution gene pool (planned)
  mcp/               — MCP server for external CC agents
```

## Recent Changes (2026-04-13)

- **Conductor architecture designed** — full agent design: runtime, tools (internal + MCP), async ops, memory/RAG, task management, channel adapters, CC workspaces. See [docs/conductor-architecture.md](docs/conductor-architecture.md)
- **MAP/COLLECT nodes** — dynamic fan-out over runtime arrays with parallel execution, scoped context isolation, `{{item.field}}` templates, custom `item_variable`, progress events
- **TRANSFORM upgrade** — from stub to LLM structured output with `output_schema` and best-effort JSON parsing
- **LoopConfig wiring** — `budget_per_iteration`, `total_budget`, loop iteration context injection, REVISION PASS/LOOP ITERATION prompt injection
- **Patch-based editing** — generator outputs `<patch>` operations for modifications (~13x fewer tokens vs full graph)
- **CC interactive sessions** — `-p` flag fix, AskUserQuestion auto-dismiss handling, tool context capture for answer workflows, `--resume` crash recovery, `--disallowed-tools`
- **Auto context_mode** — media capabilities (image_gen, video_gen, tts, stt) auto-default to "direct"
- **UI** — MAP iteration cards with progress bar, collapsible graph/patch in chat bubbles, SSE fix for child workflows, "Response:" label
- **Dev server** — workspace isolation (auto-fix editable install, auto-symlink .env)
- **Generator** — taught about all new features (MAP/COLLECT, loops, gates, disallowed_tools, output_schema)

**Deleted (PR #17):** `worker.py`, `cli/main.py`, `workflows/`, `activities/` — old Temporal code removed.

## Key Design Decisions

1. **Workflows are the universal unit** — everything composes as workflows
2. **Intelligence in evaluation, determinism in routing** — LLMs judge, code routes
3. **Protocol-based capabilities** — new provider = new file, zero executor changes
4. **Multi-model by default** — 42 models, 6 providers, automatic fallback
5. **Chat is the product** — Conductor agent is the primary interface
6. **Observability is non-negotiable** — match Claude Code desktop-level detail
7. **Fast iteration** — nuke and redeploy until PMF

## Node Types

- `execute` — LLM API call (with provider/model) or CC CLI agent (with skill_ids)
- `gate` — Routes conditionally. mode="llm" (subjective) or mode="rule" (deterministic). Fuzzy synonym matching.
- `split/join` — Static parallel branches (fixed at design time)
- `map/collect` — Dynamic fan-out from runtime arrays. Parallel iterations with scoped context, provenance tracking. Custom `item_variable` for templates.
- `transform` — LLM structured output for data reshaping. Supports `output_schema` for reliable JSON.
- `subgraph` — Calls another saved workflow by ID
- `evaluate/consensus` — Single-model and multi-model evaluation
- `human` — Pause for human input (auto-approved currently, real UI planned)

Not yet built:
- CC supervision (multi-model review loop) — designed, see [docs/cc-supervision-design.md](docs/cc-supervision-design.md)

## CC Interactive Sessions

Execute nodes can run CC CLI in interactive mode with bidirectional Q&A:

- **`interaction_mode: "workflow"`** — CC's AskUserQuestion calls are intercepted. The question is routed to an `answer_workflow_id` (e.g., a 3-model consensus workflow). The answer is sent back to CC as a user message. CC continues its work.
- **`interaction_mode: "human"`** — CC's questions are surfaced to the UI for human response (planned).
- **`max_interactions`** — safety cap on Q&A round-trips (default 10).
- **`disallowed_tools`** — list of CC tools to block via `--disallowed-tools` flag.

**How it works internally:**
1. CC is launched with `-p --input-format stream-json --output-format stream-json`
2. AskUserQuestion auto-dismisses in `-p` mode (CC says "question wasn't captured")
3. CC stays alive, waiting for the next user message
4. Engine detects AskUserQuestion in the stream, extracts the question
5. Answer workflow runs with full context (original task, conversation history, tool context)
6. Answer is sent back as a regular user message (not tool_result)
7. Loop continues until CC finishes or max_interactions reached

**Tool context:** The session captures all tool_use/tool_result events CC emits (files read, commands run, searches). This is compiled into a summary and passed to the answer workflow so it has full visibility into what CC has explored.

**Crash recovery:** If CC exits unexpectedly, the engine uses `--resume <session_id>` to restart CC with full context restoration.

## Provider Capabilities

| Provider | Chat | Vision | Image Gen | TTS | STT | Embedding | Video | Structured |
|----------|------|--------|-----------|-----|-----|-----------|-------|------------|
| Anthropic | Claude Opus/Sonnet/Haiku | Yes | — | — | — | — | — | Yes |
| OpenAI | GPT-4o/4o-mini/o3-mini | Yes | DALL-E 3, GPT-Image-1 | tts-1/hd | Whisper | text-embedding-3 | — | Yes |
| Google | Gemini 2.5 Pro/Flash/2.0 | Yes | Imagen 4, Gemini Flash Image | — | — | — | — | Yes |
| xAI | Grok 4/3 | Yes | Grok Imagine | — | — | — | — | — |
| DeepSeek | V3, Reasoner | — | — | — | — | — | — | — |
| Replicate | — | — | Flux, SDXL, SD3.5, Recraft, Ideogram + 8 more | — | — | — | Minimax, HunyuanVideo, Luma Ray | — |

## gstack Skills

gstack gives you structured workflows. Invoke with `/skill` syntax.

### Available Skills

| Skill | Purpose |
|---|---|
| `/browse` | Web browsing (always use this instead of mcp chrome tools) |
| `/investigate` | Deep-dive debugging — root cause before fixing |
| `/autoplan` | Auto-generate a plan with full review pipeline |
| `/qa` | Full QA pass — build + test + browse |
| `/review` | Pre-landing code review |
| `/ship` | Full ship flow — review, test, commit, push, PR |
| `/design-html` | Generate or iterate on HTML/CSS designs |
| `/office-hours` | Open-ended problem solving / Q&A session |

### Proportionality — match tool weight to task weight

| Task size | What to do |
|---|---|
| **Trivial** (≤2 files, obvious fix) | Just fix it. No skills needed. |
| **Small** (single issue, clear approach) | `/ship` to create the PR. |
| **Medium** (multi-file, any UI change) | `/review` → `/ship`. Add `/qa` if UI changed. |
| **Large** (architectural, 10+ files) | `/autoplan` → implement → `/qa` → `/review` → `/ship` |

### Do not use these skills (require human in the loop)

`/office-hours`, `/design-shotgun`, `/design-consultation`, `/connect-chrome`, `/setup-browser-cookies`, `/retro`

## Skill Routing

When the user's request matches an available skill, ALWAYS invoke it using the Skill
tool as your FIRST action. Do NOT answer directly, do NOT use other tools first.

Key routing rules:
- Bugs, errors, "why is this broken" → invoke investigate
- Ship, deploy, push, create PR → invoke ship
- QA, test the site, find bugs → invoke qa
- Code review, check my diff → invoke review
- Architecture review → invoke plan-eng-review

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joelgreen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
