## ultimatrix

> - **1761 tests (169 files), clean tsup build (ESM 1.61MB + CJS 1.63MB + DTS)**

## Ultimatrix v8 — Intelligence-Augmented Security Researcher

### Status
- **1761 tests (169 files), clean tsup build (ESM 1.61MB + CJS 1.63MB + DTS)**
- **318 source files**, zero test failures
- **Dual engine**: Legacy supervisor (v6/v7) + OODA solver engine (v8)
- **Council engine**: Parallel debate with structured typed outputs (no regex/text parsing)
- **56 skills** (10 domains), knowledge-based, not payload lists
- **Skill-driven tool filtering**: Skills declare toolRefs in YAML frontmatter, tools filtered per-agent
- **Human-in-the-Loop**: Browser visibility, action capture, session storage, flow reproduction
- **FIX-PLAN v8.2 COMPLETED**: All root-cause fixes implemented and verified
- **Council root-cause rewrite COMPLETED**: All regex removed, structured typed fields at all seams
- `@mastra/core` ^1.42.0, `playwright` ^1.52.0, `zod` ^4.0.0, `next` ^15.5.19

### Architecture — Dual Engine + Council

```
                    ┌──────────────────────┐
                    │   Engine Selector    │ ← config.engine: 'legacy' | 'solver'
                    └──────────┬───────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                 ▼
    ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐
    │  Legacy      │  │  Solver      │  │  Council         │
    │  Supervisor  │  │  Engine      │  │  Engine          │
    │  (Phase 1-5) │  │  (OODA)      │  │  (parallel       │
    │              │  │              │  │   debate)        │
    │  observe →   │  │  REASON →    │  │                  │
    │  learn →     │  │  EXPLORE →   │  │  debateOnce()    │
    │  attack →    │  │  CONCLUDE →  │  │  4 LLM members   │
    │  loop        │  │  loop        │  │  parallel speaks  │
    └──────────────┘  └──────────────┘  └──────────────────┘
              │                │                 │
              └────────────────┼────────────────┘
                               ▼
                     ┌──────────────────────┐
                     │  Session Runner      │ ← src/session.ts
                     │  (CoreServices)      │   ExecutionStrategy interface
                     └──────────┬───────────┘
                               │
                    ┌──────────┴───────────┐
                    ▼                      ▼
           ┌──────────────┐      ┌──────────────────┐
           │  Skills Lib  │      │  Skill-Tool      │
            │  (56 skills) │      │  Filter          │
           │  YAML meta   │      │  core tools      │
           └──────────────┘      │  always included │
                                 └──────────────────┘
```

### Execution Core (`src/core/`)

Unified interface for all engines. Each engine implements `ExecutionStrategy`.

| Module | Location | Purpose |
|--------|----------|---------|
| **Types** | `src/core/types.ts` | `ExecutionStrategy`, `StrategyContext`, `CoreServices`, `EnginePreset`, `RunResult` |
| **ToolPack** | `src/core/toolpack.ts` | Shared tool-pack builder for brain + council (core, http, skill, research, orchestration) |
| **Evidence** | `src/core/evidence.ts` | Shared singleton `EvidenceLedger` for all tools + gate + council |
| **Blackboard** | `src/core/blackboard.ts` | Unified fact/intent state-space + Plan model + tool-call dedup |
| **Approval** | `src/core/approval.ts` | Re-exports council approval as shared gate for both strategies |

### Council Engine (`src/council/`)

Parallel debate: 4 LLM members (strategist, operator, skeptic, analyst) debate what to test, analyze results together. Human steers via HITL approval.

| Module | Location | Purpose |
|--------|----------|---------|
| **Types** | `src/council/types.ts` | `MemberOutput`, `CouncilIntent`, `ImpactLevel`, `CouncilProposal`, `CouncilCritique`, `CouncilReflection`, `DebateCycleResult` |
| **Personas** | `src/council/personas.ts` | LLM persona strings per role + mandatory structured output contract (JSON block) |
| **Factory** | `src/council/factory.ts` | Builds real LLM-backed council agents, `parseStructuredOutput()` extracts typed JSON |
| **Orchestrator** | `src/council/orchestrator.ts` | `debateOnce()` (parallel, one cycle), `runCouncil()` (backward-compat loop) — zero regex |
| **Bus** | `src/council/bus.ts` | `ConversationBus`: append-only log + sliding-window transcript for members |
| **Blackboard** | `src/council/blackboard-shared.ts` | `SharedBlackboard` adapter wrapping core `Blackboard` for council |
| **Evidence Bridge** | `src/council/evidence-bridge.ts` | `bridgeWorkerToolCall()`, `bridgeWorkerEvidence()`, `extractProposedTasks()` — structured extraction |
| **Approval** | `src/council/approval.ts` | `classifyImpact()` reads typed `proposal.impact` field, zero regex. HITL gate for critical. |

**Design principle:** No hardcoded substring detection. Structured typed fields at all seams. All regex removed from council code.

### Intelligence Layer (`src/intelligence/`)

| Module | Location | Purpose | Tests |
|--------|----------|---------|-------|
| **Evidence Gate** | `src/intelligence/evidence-gate.ts` | Anti-hallucination: cross-checks LLM claims against typed evidence | 14 |
| **Evidence Ledger** | `src/intelligence/evidence-ledger.ts` | Structured `ObservedFacts`, `EvidenceItem`, `FindingClaim`, `verifyFindingClaim` | 9 |
| **Reflexion Engine** | `src/intelligence/reflexion.ts` | Failure classification, L0-L4 escalation, experience extraction | 25 |
| **Anti-Loop** | `src/intelligence/anti-loop.ts` | Stale/dead-end detection, structured `[PATH:]` extraction | 20 |
| **Constants** | `src/intelligence/constants.ts` | Centralized signal lists (no keyword duplication) | — |
| **Reflexion Store** | `src/intelligence/reflexion-store.ts` | Persist/load reflexion state to graph | 12 |
| **Cross-Engagement** | `src/intelligence/cross-engagement.ts` | Privacy-preserving cross-session pattern memory | — |
| **Outcome Feedback** | `src/intelligence/outcome-feedback.ts` | Post-engagement feedback loop: finding acceptance → technique weights | — |
| **Chaining** | `src/intelligence/chaining.ts` | Finding chain detection: links findings into multi-step attack chains | 12 |
| **Hypotheses** | `src/intelligence/hypotheses.ts` | Attack hypothesis generator from graph state | 9 |
| **Auth Recorder** | `src/intelligence/auth-recorder.ts` | Auth flow recorder: login, OAuth, SAML → AuthFlow nodes | 18 |
| **RBAC Learner** | `src/intelligence/rbac-learner.ts` | RBAC learner: role-based access patterns → RBACMatrix nodes | — |
| **Session Resume** | `src/intelligence/session-resume.ts` | Previous session detection + continuity | — |

### Solver Engine (`src/solver/`)

OODA loop: REASON → EXPLORE → CONCLUDE with intelligence layers observing passively.

| Module | Location | Purpose |
|--------|----------|---------|
| **Solver** | `src/solver/solver.ts` | Main OODA loop: `agent.stream()` per REPL turn, intelligence layers observe |
| **Blackboard** | `src/solver/blackboard.ts` | Re-exports core `Blackboard` for backward compatibility |
| **Brain Tools** | `src/solver/brain-tools.ts` | Solver brain agent with focused ~30 tool set |
| **Brain Instructions** | `src/solver/brain-instructions.ts` | Persona instructions: role, safety, workflow, anti-hallucination rules |
| **Attack Path** | `src/solver/attack-path.ts` | Chains endpoint+finding steps into exploit paths on graph |
| **Plan Tools** | `src/solver/plan-tools.ts` | Structured planning tools: `createPlan`, `updatePlan`, `getPlan` |

**Skill Subsystem (`src/solver/skills/`):**

| Module | Location | Purpose |
|--------|----------|---------|
| **Tool Filter** | `src/solver/skills/tool-filter.ts` | `resolveToolsForSkills()` + `CORE_TOOLS` negative scoring |
| **Loader** | `src/solver/skills/loader.ts` | YAML frontmatter parsing, skill body loading, reference extraction |
| **Registry** | `src/solver/skills/registry.ts` | Graph-aware skill selection by endpoint type, auth, technique history |
| **Technique Registry** | `src/skills/technique-registry.ts` | Single source of truth for attack techniques, tool mappings, chain rules |

### Skills Library (56 skills, 10 domains)

Skills live at `skills/` (project root). YAML frontmatter declares metadata, toolRefs, and composition rules.

| Domain | Skills |
|--------|--------|
| Injection | exploitation, vuln-discovery, nosql-injection, second-order-sqli, ssti, xxe, command-injection-advanced, email-injection |
| Web Attacks | web-pentest, web-security-advanced, waf-bypass, blind-ssrf, business-logic, race-conditions-advanced, http-smuggling, deserialization, cors-misconfig, clickjacking, cache-poisoning, open-redirect, prototype-pollution, host-header-injection, css-injection, file-upload-attacks, type-juggling, modern-xss, security-headers-audit |
| Auth Security | authorization, jwt-advanced, jwt-algorithm-confusion |
| Recon | recon, osint-recon, information-disclosure, intranet-pentest, post-exploitation, subdomain-takeover, hsts-bypass, ssl-stripping, ctf-misc |
| Crypto | crypto-toolkit, ctf-crypto |
| API Security | api-security, api-fuzzing, ai-mcp-security, graphql-attacks, graphql-depth-introspection, websocket-attacks |
| Cloud Security | aws-iam-exploitation, azure-exploitation, gcp-exploitation, docker-escape, kubernetes-security, serverless-attacks |
| LLM Security | llm-agentic-security |
| Supply Chain | supply-chain |
| Reports | reporting |

### Campaign Engine (`src/campaign/`)

Autonomous coverage planning: builds endpoint × param × role × state matrix, dedupes into slices, bounded-concurrency execution.

| Module | Location | Purpose |
|--------|----------|---------|
| **Types** | `src/campaign/types.ts` | `CampaignSlice`, `CoverageStats`, `PrimitiveRef`, `CampaignPlan` |
| **Planner** | `src/campaign/planner.ts` | Builds coverage matrix, dedupes into executable slices |
| **Executor** | `src/campaign/executor.ts` | Rate limiting, budget guard, bounded-concurrency slice execution |
| **Runner** | `src/campaign/runner.ts` | Resolves primitive by id, runs via HTTP tool, records into EvidenceGate |
| **Continuity** | `src/campaign/continuity.ts` | App-state hashing, change detection, incremental re-test planning |
| **Campaign Tool** | `src/campaign/campaign-tool.ts` | Mastra dispatch tool: lets LLM strategist emit whole campaigns |

### Analysis Pipeline (`src/analysis/`)

HAR-based business-logic analysis: value-provenance graph, auth decode, custom headers, invariants.

| Module | Location | Purpose |
|--------|----------|---------|
| **Analyser** | `src/analysis/analyser.ts` | Business-logic analyser: value-provenance graph, auth decode, invariants from HAR |
| **HAR Bridge** | `src/analysis/har-bridge.ts` | Wires HAR pipeline into graph + LLM context |
| **HAR Analyzer** | `src/analysis/har-analyzer.ts` | HAR analysis: extracts endpoints, secrets, data flows, patterns |
| **Skill Loader** | `src/analysis/skill-loader.ts` | Analysis skill loader: reads .md skill files, categorizes |
| **Instructions** | `src/analysis/instructions.ts` | Builds LLM instructions from skills + HAR data + target context |

### Capture & Browser (`src/capture/`, `src/browser/`)

| Module | Location | Purpose |
|--------|----------|---------|
| **Human Observer** | `src/capture/human-observer.ts` | Playwright event hooks: captures human click/fill/navigate + `AuthStateDetector` |
| **HAR Parser** | `src/capture/har-parser.ts` | HAR 1.2 parser with Zod schemas: endpoints, secrets, data flows |
| **Graph Bridge** | `src/capture/graph-bridge.ts` | Persists Stagehand tool results to graph as nodes |
| **Browser Launcher** | `src/capture/browser-launcher.ts` | Playwright browser launcher with managed page lifecycle |
| **Network Capture** | `src/capture/network-capture.ts` | Playwright network interceptor: requests/responses → HAR entries |
| **Passive Observer** | `src/capture/passive-observer.ts` | Passive DOM observer: watches XHR/fetch patterns, writes to graph |
| **Anti-Bot** | `src/browser/anti-bot.ts` | Bot detection: Cloudflare/Akamai/DataDome/PerimeterX, 30s wait |
| **Browser Manager** | `src/browser/manager.ts` | Singleton StagehandBrowser, `getActivePage()`, screenshot capture |
| **State Bridge** | `src/browser/state-bridge.ts` | Imports/exports cookies+storage into Stagehand context |
| **Dialog Watcher** | `src/browser/dialog-watcher.ts` | CDP-level dialog detection (alert/confirm/prompt), auto-dismiss |
| **Dialog Inject** | `src/browser/dialog-inject.ts` | Wraps Stagehand tools to auto-inject dialog evidence |
| **Reaction Observer** | `src/browser/reaction-observer.ts` | Post-action DOM diffing: modals, toasts, errors, success messages |

### Tools (`src/tools/`)

| Module | Location | Purpose |
|--------|----------|---------|
| **Encode/Decode** | `src/tools/encode-decode.ts` | Base64/URL/HTML encode/decode |
| **Skill Tools** | `src/tools/skill-tools.ts` | `listSkills` + `loadSkillReference` + `searchSkills` |
| **Flow Tools** | `src/tools/flow-tools.ts` | `saveSession`, `restoreSession`, `observeHumanActions`, `saveLearnedFlow`, `reproduceFlow` |
| **Interaction** | `src/tools/interaction-tools.ts` | `askUser` with `waitForBrowserAction` + screenshot capture |
| **Browser Auth** | `src/tools/extract-browser-auth.ts` | Extract auth tokens from browser (localStorage, sessionStorage, cookies) |
| **Budget Dashboard** | `src/tools/budget-dashboard.ts` | Aggregates forensic log events into session-level budget/usage summaries |
| **Budget Pruner** | `src/tools/budget-pruner.ts` | Token-budget-aware tool pruning under budget pressure |
| **Control Tools** | `src/tools/control-tools.ts` | `writeFinding`, `updateGraph`, `recordTest` — core evidence + graph mutation |
| **Recon Tools** | `src/tools/recon-tools.ts` | WHOIS, DNS, subdomain brute-force, JWT decode, scope-guard-aware recon |
| **Token Profiler** | `src/tools/token-profiler.ts` | Per-tool token usage tracking |
| **Tool Registry** | `src/tools/registry.ts` | `registerAllTools()` — central registry of all Mastra tools |
| **Tool Availability** | `src/tools/tool-availability.ts` | `isToolAvailable()` — checks if a tool binary exists on PATH |

### Other Modules

| Module | Location | Purpose |
|--------|----------|---------|
| **Forensic Log** | `src/logging/forensic-log.ts` | NDJSON forensic event logging (tool-call, http-request, graph-mutation, model-call) |
| **System Logger** | `src/logging/system-logger.ts` | System metrics: compression, dialog, memory stats |
| **Compression** | `src/compression/headroom-service.ts` | `CompressionService` using headroom-ai: structured `CompressionResult` |
| **Scope Guard** | `src/safety/scope-guard.ts` | URL scope enforcement: deny-by-default, `isUrlInScope()` |
| **Core Contract** | `src/prompts/core-contract.ts` | Anti-hallucination, workflow rules, PATH declarations (English, ~300 words) |
| **Rate Limiter** | `src/models/rate-limiter.ts` | Sliding window + cooldown rate limiting |
| **Quota Tracker** | `src/models/quota-tracker.ts` | Provider quota exhaustion tracking with cooldown |
| **Model Selector** | `src/models/selector.ts` | `selectForTask()` scores models by complexity/capabilities/rate-limit/exhaustion |
| **Model Middleware** | `src/models/middleware.ts` | `wrapModel()` rate limiting Proxy |
| **Schema Sanitizer** | `src/models/schema-sanitizer.ts` | Provider-compatible JSON Schema |
| **Context Manager** | `src/models/context-manager.ts` | Context window overflow management |
| **Token Budget** | `src/models/token-budget-tracker.ts` | Session-level token budget tracking |

### Legacy Architecture (v6, still present)

- **Supervisor Agent** (`src/manager/agent.ts`): Mastra Agent — Observe-Learn-Attack loop
- **4 Specialist Workers** (`src/workers/`): `injection`, `authControl`, `advanced`, `recon`
- **Spider Agent** (`src/spider/agent.ts`): Stagehand-based hybrid crawler
- **Action Recorder** (`src/recorder/`): Browser actions → test cases → Playwright code
- **OAST Server** (`src/oast/`): Blind callback detector
- **AgentManager** (`src/lib/agent-manager.ts`): Singleton — owns browser, workers, supervisor
- **Web UI** (`src/app/`, `src/components/`): Next.js 15 + shadcn/ui interface

### CLI Commands

| Command | Description |
|---------|-------------|
| `ultimatrix init` | Interactive provider + config setup wizard |
| `ultimatrix solve -t <url>` | OODA solver engine against target |
| `ultimatrix interact -t <url>` | Terminal REPL (legacy supervisor or solver per config) |
| `ultimatrix scan -t <url>` | Full scan: learn + generate + report |
| `ultimatrix learn -t <url>` | Capture traffic, parse HAR, analyze patterns |
| `ultimatrix generate -t <url>` | Learn → generate Playwright test cases |
| `ultimatrix replay` | Re-run previously generated tests |
| `ultimatrix report` | Generate JSON/HTML/Markdown report |
| `ultimatrix web` | Next.js web UI (legacy v6) |
| `ultimatrix assess -t <url>` | Full assessment (legacy v6) |
| `ultimatrix verify -a <model> -t <url>` | Re-run findings (legacy v6) |

### Engine Routing

- `config.engine: 'multi-model'` (default) — Solver + model-aware delegation
- `config.engine: 'legacy'` — Uses supervisor + 4 worker agents
- `config.engine: 'solver'` — Uses OODA solver loop (REASON → EXPLORE → CONCLUDE)
- `config.engine: 'council'` — Parallel debate with HITL (on-demand)
- `ultimatrix solve` always uses solver engine
- `ultimatrix interact` respects `config.engine`

### Graph Schema (24 node types, 20 edge types)

**Node types:** Page, Action, Input, Endpoint, Test, Finding, AuthFlow, RBACRole, Attack, Fact, Intent, Reflexion, Workflow, Entity, Hypothesis, Experiment, CandidateFinding, HeaderSemantic, AuthScheme, OutcomeFeedback, RenderedElement, CouncilDebate, ExploitProof, ThreatModel

**Edge types:** HAS_ACTION, HAS_INPUT, HAS_TEST, FOUND_ON, REQUIRES_AUTH, CHAINED_FROM, TARGETS, PRODUCED, HAS_ROLE, PERMISSION, BUILT_ON, PRODUCED_BY, VALUE_ORIGIN, REQUIRES_ROLE, CHAINS_TO, RENDERED_ON, REINGESTS, ORDERED_BEFORE, PROVES, SESSION_REACHES

### Config

```yaml
provider: groq          # or openai, anthropic, google, nvidia, etc.
model: llama3-8b-8192
target: https://example.com
engine: solver          # 'legacy' | 'solver'
solver:
  maxToolCalls: 50      # Max tool-call rounds per turn
  maxTokens: 100000     # Max tokens per turn
  maxDurationMs: 300000 # Max wall-clock time per turn
  maxParallel: 1
antiLoop:
  staleThreshold: 3
reflexion:
  persistToGraph: true
```

### Key Commands

- `npm test` — full test suite (1761/1761)
- `npm run build:cli` — tsup build
- `npm run lint` — eslint src/
- `npx ultimatrix solve -t <url>` — OODA solver
- `npx ultimatrix interact -t <url>` — terminal REPL

### Known Issues

- Legacy v6 modules (`src/context/`, `src/lib/agent-manager.ts`) have type errors — pre-existing tech debt, not blocking v8
- Cloudflare challenges block Stagehand crawl — deferred
- ESLint configured (`eslint.config.js`) but `npm run lint` times out — needs rule tuning for large codebase

### Skills (project)
- **customize-opencode** — Editing opencode's own configuration files only

---
> Source: [Msalways/Ultimatrix](https://github.com/Msalways/Ultimatrix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
